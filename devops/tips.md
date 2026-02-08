# Production Survival Guide

The stuff you only really learn **after** breaking prod at least once.

**Table of contents**

1. [The Bottleneck Trap](#1-the-bottleneck-trap)
2. [The Silent Killer](#2-the-silent-killer)
3. [Pod Lifecycle](#3-pod-lifecycle)
4. [The Probe Mistake](#4-the-probe-mistake)
5. [IaC Danger](#5-iac-danger)
6. [Terraform Reality](#6-terraform-reality)
7. [Scaling Strategy](#7-scaling-strategy)
8. [The "Green" Deployment](#8-the-green-deployment)
9. [Incident Response](#9-incident-response)
10. [The Observability Trinity](#10-the-observability-trinity)

---

## At a glance

| Situation                   | First check                   | Don’t do           |
| --------------------------- | ----------------------------- | ------------------ |
| High latency, CPU idle      | `iostat -x`, `%wa`            | Add more CPU       |
| Pods restarting in a loop   | Readiness vs liveness logic   | Add more replicas  |
| “We fixed prod manually”    | `terraform plan`              | `terraform apply`  |
| Deployment green, users 502 | Warmup, migrations, LB health | Trust the pipeline |
| Active outage               | Roll back                     | Hotfix in prod     |

---

## 1. The Bottleneck Trap

### CPU-bound vs I/O-bound

- **CPU-bound**
  - The process is busy _computing_
  - More cores = faster

- **I/O-bound**
  - The process is mostly _waiting_ (disk, network, DB)

**The command that proves it**

```bash
top   # or htop
```

- High CPU %, low I/O wait → CPU-bound
- Low CPU %, high `%wa` → I/O-bound

**Truth serum:**

```bash
iostat -x
```

High `await` = I/O bottleneck.

> If CPU isn’t busy but latency is high, you’re waiting on something else.

---

## 2. The Silent Killer

### Why servers crash when CPU & RAM look fine

Because **something else hit a hard limit**:

- File descriptors (`ulimit -n`)
- Ephemeral disk
- Kernel memory
- TCP connection tracking
- Thread pools

The OS doesn’t negotiate.
OOM is loud. **Resource exhaustion is quiet and lethal.**

---

## 3. Pod Lifecycle

### Pod restart vs Node failure

**Pod restart (same node)**

- Container filesystem — wiped
- PVC volumes — survive
- Memory — gone

**Node failure**

- Pod rescheduled elsewhere
- `emptyDir` — gone
- PVC — reattached
- Local state — dead forever

If data matters, it doesn’t belong in the Pod.

---

## 4. The Probe Mistake

### Readiness vs Liveness

- **Readiness** — “Can I receive traffic?”
- **Liveness** — “Should Kubernetes kill me?”

**The classic outage pattern:**

- Liveness checks DB
- DB slows
- Pods restart
- Thundering herd
- Full outage

**Rule of thumb**

> If a restart won’t fix it, it’s **NOT** liveness.

---

## 5. IaC Danger

### When Infrastructure as Code bites back

IaC becomes dangerous when:

- You don’t understand the blast radius
- State is outdated
- Changes are destructive (`force_new_resource`)
- Prod was “fixed” manually

IaC doesn’t ask _should we_.
It only asks _can we_.

---

## 6. Terraform Reality

### How State _actually_ works

Terraform State is the **source of truth**, not reality.

- Manual change → Terraform doesn’t know
- `terraform plan` → shows drift
- `terraform refresh` → updates state only
- `terraform import` → adopts unmanaged resources

**Golden rule:**

- Minor drift → **refresh**
- Unmanaged resource → **import**
- Blind apply → **career-limiting event**

---

## 7. Scaling Strategy

### When to AVOID horizontal autoscaling

Never autoscale when:

- The bottleneck is a **shared dependency** (DB, lock, queue)
- Startup time is slow
- Licensing or cost explodes
- The app is not stateless

Scaling broken systems just makes them fail faster.

---

## 8. The “Green” Deployment

### Why pipelines pass but users see 502s

“Green” means:

- Pods are running
- Containers started

It does **NOT** mean:

- App is warmed up
- Migrations finished
- Cache primed
- Load balancer health aligned

CI/CD checks _deployment success_.
Users care about _request success_.

---

## 9. Incident Response

### Hotfix vs Rollback

**Rollback. Always.**

Hotfixes are for:

- Calm environments
- No active outage
- Plenty of time

During an incident:

1. Roll back to last known good
2. Stop the bleeding
3. Fix forward later

Hero hotfixes extend outages.

---

## 10. The Observability Trinity

### Metrics, Logs, Traces

**During an outage, look in this order:**

1. **Metrics** — _What_ is broken?
2. **Logs** — _Why_ is it broken?
3. **Traces** — _Where_ exactly is it broken?

Start with metrics, not logs — otherwise you’re guessing. Metrics give you the map.

**Quick checks:**

- **Metrics:** dashboards → compare before/after, check error rate and latency percentiles.
- **Logs:** filter by time window and level; search for exceptions and stack traces.
- **Traces:** follow a failing request end-to-end; find the slow or failing span.

---

## See also

- [DevOps README](./README.md) — index of all DevOps notes in this repo.
- Use this doc as a pre-deploy checklist and incident runbook reminder.
