# SRE Demo - Presenter's Guide

## Reducing MTTR with Cortex.io

**Demo Duration:** 15-20 minutes  
**Star of the Show:** Cortex.io  
**Supporting Cast:** Dynatrace, PagerDuty

---

## Executive Summary

This demo showcases how **Cortex.io** dramatically reduces Mean Time To Resolution (MTTR) by providing on-call engineers with instant access to:
- Service dependencies and architecture
- Runbooks and remediation steps  
- Service owners and escalation paths
- All in one centralized location

**The Story:** An unfamiliar on-call engineer receives an alert at 2 AM. Without tribal knowledge, they use Cortex.io to quickly understand the system, identify the root cause, find the fix, and resolve the incident—all within minutes.

---

## Pre-Demo Setup

### Checklist
- [ ] All 5 services running in EKS (`kubectl get pods`)
- [ ] Frontend accessible via LoadBalancer URL
- [ ] Dynatrace showing all services healthy
- [ ] PagerDuty integration configured
- [ ] Cortex.io service catalog populated with all services
- [ ] Browser tabs pre-opened:
  - Tab 1: Frontend application
  - Tab 2: Dynatrace (Services view)
  - Tab 3: PagerDuty (Incidents)
  - Tab 4: Cortex.io (Service Catalog)
- [ ] Terminal ready for kubectl commands

### Pre-Load Baseline (Optional)
Send a few orders before the demo so Dynatrace has baseline metrics:
```bash
# 5 minutes before demo, send light traffic
for i in {1..20}; do
  curl -s -X POST http://<FRONTEND_URL>/api/orders \
    -H "Content-Type: application/json" \
    -d '{"productId":"PROD-001","quantity":1}' > /dev/null
  sleep 5
done
```

---

## Demo Script

### ACT 1: Set the Scene (2 minutes)

#### Talking Points:
> "Imagine it's 2 AM. You're the on-call engineer this week—but you just joined the team last month. You're not familiar with all these microservices yet."
>
> "Let's look at what we're dealing with: a typical e-commerce platform with 5 microservices."

#### Action: Show the Frontend
1. Open the Frontend application
2. Briefly explain the architecture diagram on the UI:
   - "Frontend talks to Order Service"
   - "Order Service calls Inventory Service"
   - "And so on down the chain"

> "Everything looks healthy right now. But that's about to change."

---

### ACT 2: Trigger the Incident (2 minutes)

#### Talking Points:
> "We just launched a flash sale. Traffic is spiking."

#### Action: Generate Load
1. On the Frontend, set:
   - Requests/sec: **20**
   - Duration: **120 seconds**
2. Click **"Start Continuous Load"**

> "Watch the average latency in the stats panel... it's climbing."
>
> "Under normal conditions, orders complete in under 200 milliseconds. Now we're seeing 1, 2, 3 seconds... and climbing."

#### Action: Show Initial Impact
- Point to the **Avg Latency** stat rising
- Point to the **Success Rate** potentially dropping
- Note the log entries showing slow responses

> "Users are experiencing this right now. Shopping carts are timing out. We're losing revenue every second."

---

### ACT 3: Dynatrace Detects the Problem (2 minutes)

#### Talking Points:
> "Fortunately, we have Dynatrace monitoring our environment. It's already detected something is wrong."

#### Action: Switch to Dynatrace
1. Navigate to **Problems** or **Services → frontend**
2. Show the response time degradation alert

> "Dynatrace has detected response time degradation on our frontend service. That's where users are experiencing the slowness."
>
> "But here's the challenge: the alert is on the frontend—but is that where the actual problem is?"

#### Key Point:
> "This is where most teams start the war room. They pull in the frontend team, start digging through logs, maybe blame the CDN... This can take **30 minutes to hours** just to find the right team to engage."

---

### ACT 4: PagerDuty Alert (1 minute)

#### Talking Points:
> "Dynatrace has automatically sent an alert to PagerDuty, and our on-call engineer just got paged."

#### Action: Switch to PagerDuty
1. Show the incident in PagerDuty
2. Highlight the alert details (service name, problem description)
3. **Point to the Cortex.io link** in the incident

> "Now, our on-call engineer—remember, they're new to the team—sees this alert. They could start SSH-ing into servers, or they could click this link to Cortex."
>
> "Let's see what happens when they choose Cortex."

---

### ACT 5: Cortex.io - The Hero Moment (5-7 minutes)

This is the star of the show. Take your time here.

#### 5A: Service Overview

**Action:** Click the Cortex.io link → lands on frontend service page

> "Immediately, our engineer sees everything they need to know about this service."

**Highlight:**
- Service description
- Owner team (Customer Experience)
- Slack channel (#cx-team)
- On-call contact (@cx-oncall)

> "Without Cortex, finding this information might require digging through Confluence, Slack, or asking around. That's 10-15 minutes gone. With Cortex, it's **instant**."

---

#### 5B: Dependency Discovery ⭐ KEY MOMENT

**Action:** Navigate to the Dependencies view in Cortex

> "But here's where Cortex really shines. Our engineer looks at the **dependencies** of the frontend service."

**Show the dependency graph:**
```
frontend → order-service → inventory-service → payment-service → notification-service
```

> "The frontend depends on order-service, which depends on inventory-service. Now our engineer can trace the request path."
>
> "Instead of guessing, they can systematically check each service."

**Action:** Click on **inventory-service** in the dependency graph

> "Let's look at inventory-service..."

---

#### 5C: Identifying the Root Cause

**Action:** On the inventory-service page in Cortex, highlight:
- Different owner team (**Platform Infrastructure**)
- Different Slack channel (**#platform-eng**)
- Different on-call (**@platform-oncall**)

> "Immediately, our engineer sees this is owned by a **different team**. Without Cortex, they might have spent 20 minutes in the wrong Slack channel."

**Highlight the Runbooks section:**

> "And look at this—there's a **runbook** linked directly in Cortex: 'CPU Throttling Remediation.'"
>
> "Our new on-call engineer doesn't need to have tribal knowledge. The answer is right here."

---

#### 5D: The Runbook ⭐ KEY MOMENT

**Action:** Click to open the runbook

> "The runbook tells them exactly what's happening and how to fix it."

**Walk through the runbook sections:**

1. **Symptoms:** "Response time degradation—check."
2. **Root Cause:** "CPU limits too low causing throttling under load."
3. **Resolution:** "Here's the exact kubectl command to run."

> "This is the power of Cortex. A brand new engineer, at 2 AM, with no prior knowledge of this system, can resolve this incident by following documented steps."
>
> "No war rooms. No waking up senior engineers. No guessing."

---

#### 5E: Finding the Right People (if escalation needed)

**Action:** Show the owner/contact information on the inventory-service page

> "If our engineer needed to escalate—maybe the runbook didn't work—they know exactly who to contact:"
> - "Platform Infrastructure team"
> - "#platform-eng Slack channel"
> - "@platform-oncall for immediate escalation"

> "Cortex has turned a 30-minute 'who owns this?' investigation into a **5-second lookup**."

---

### ACT 6: Apply the Fix (2 minutes)

#### Talking Points:
> "Let's apply the fix from the runbook."

#### Action: Run the Fix
```bash
kubectl patch deployment inventory-service --type='json' -p='[
  {"op": "replace", "path": "/spec/template/spec/containers/0/resources/requests/cpu", "value": "500m"},
  {"op": "replace", "path": "/spec/template/spec/containers/0/resources/limits/cpu", "value": "1000m"}
]'
```

> "We're increasing the CPU limit from 200 millicores to 1 full core."

#### Action: Watch the Rollout
```bash
kubectl rollout status deployment/inventory-service
```

> "The new pod is starting with the correct resources."

---

### ACT 7: Verify Resolution (2 minutes)

#### Action: Return to Frontend
- Show latency dropping back to normal
- Show success rate recovering

> "Look at the latency—it's dropping. We're back under 200 milliseconds."

#### Action: Return to Dynatrace
- Show the problem auto-closing (or trending toward resolution)
- Show CPU throttling metric dropping

> "Dynatrace confirms the CPU throttling has stopped. The problem will auto-close shortly."

---

### ACT 8: The Wrap-Up (2 minutes)

#### Talking Points:

> "Let's recap what just happened:"
>
> "A new on-call engineer, with **no prior knowledge** of this system, was able to:"

1. ✅ "Receive an alert via PagerDuty"
2. ✅ "Instantly access service context via Cortex.io"
3. ✅ "Trace dependencies to find the root cause"
4. ✅ "Find the right team and escalation path"
5. ✅ "Access a runbook with step-by-step remediation"
6. ✅ "Resolve the incident in **under 10 minutes**"

> "Without Cortex, this same incident might have taken **30 minutes to an hour**:"
> - "10-15 minutes finding the right team"
> - "10-15 minutes tracking down documentation"
> - "10+ minutes coordinating across teams"

> "**Cortex.io reduced our MTTR by 60-70%.**"

---

## Key Value Props to Emphasize

Throughout the demo, reinforce these Cortex.io benefits:

| Traditional Approach | With Cortex.io |
|---------------------|----------------|
| "Who owns this service?" - Slack search, asking around | **Instant lookup** - owner visible on service page |
| "What does this service depend on?" - Guessing, architecture docs (if they exist) | **Dependency graph** - visual, always up-to-date |
| "Is there a runbook?" - Confluence search, tribal knowledge | **Linked directly** - one click from service page |
| "Who do I escalate to?" - Org charts, Slack hunting | **On-call info** - integrated with PagerDuty |
| "What's the Slack channel?" - Trial and error | **Listed on service page** - no guessing |

---

## Handling Questions

### "How does Cortex get this data?"
> "Teams maintain simple YAML files in their repos, or Cortex can integrate with existing tools like GitHub, PagerDuty, and Kubernetes to auto-discover services."

### "What if the runbook is out of date?"
> "Cortex integrates with your Git repos, so runbooks are version-controlled and reviewed like code. You can also set up scorecards to ensure runbooks stay current."

### "Does this work with our existing tools?"
> "Cortex integrates with Dynatrace, PagerDuty, GitHub, Kubernetes, and 50+ other tools. It's designed to be the single pane of glass that connects everything."

### "How is this different from a wiki?"
> "Wikis are static and often outdated. Cortex is a **living catalog** that pulls real-time data from your infrastructure, so ownership and dependencies are always current."

---

## Timing Summary

| Act | Content | Duration |
|-----|---------|----------|
| 1 | Set the Scene | 2 min |
| 2 | Trigger the Incident | 2 min |
| 3 | Dynatrace Detects | 2 min |
| 4 | PagerDuty Alert | 1 min |
| **5** | **Cortex.io Deep Dive** | **5-7 min** |
| 6 | Apply the Fix | 2 min |
| 7 | Verify Resolution | 2 min |
| 8 | Wrap-Up | 2 min |
| | **Total** | **18-20 min** |

---

## Post-Demo Cleanup

Reset for next demo:
```bash
# Revert to throttled state
kubectl patch deployment inventory-service --type='json' -p='[
  {"op": "replace", "path": "/spec/template/spec/containers/0/resources/requests/cpu", "value": "200m"},
  {"op": "replace", "path": "/spec/template/spec/containers/0/resources/limits/cpu", "value": "200m"}
]'

# Wait for rollout
kubectl rollout status deployment/inventory-service

# Verify
kubectl get deployment inventory-service -o jsonpath='{.spec.template.spec.containers[0].resources}'
```

---

## Appendix: Demo Environment Details

| Component | URL/Details |
|-----------|-------------|
| Frontend | a64106ff620e641809e000cfa89f908e-280535655.us-east-2.elb.amazonaws.com |
| GitHub | https://github.com/CheemaKhasham/front-end, https://github.com/CheemaKhasham/order-service, https://github.com/CheemaKhasham/inventory-service, https://github.com/CheemaKhasham/payment-service, https://github.com/CheemaKhasham/notification-service |
| Cortex.io | https://app.cortex.io/catalog |
| Dynatrace | https://crr35672.apps.dynatrace.com/ |
| PagerDuty | Your org URL |
