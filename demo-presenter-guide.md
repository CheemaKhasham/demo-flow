# SRE Demo - Presenter's Guide

## Reducing MTTR with Cortex.io

**Demo Duration:** 20-25 minutes  
**Star of the Show:** Cortex.io  
**Supporting Cast:** Dynatrace, PagerDuty, Jira

---

## Executive Summary

This demo showcases how **Cortex.io** dramatically reduces Mean Time To Resolution (MTTR) by providing on-call engineers with instant access to:
- Service scorecards and health indicators
- Visual dependency graphs
- Recent deployments and changes
- Previous incidents and patterns
- Runbooks and remediation steps
- Service owners and escalation paths
- Post-incident workflows (Jira integration)

**The Story:** An unfamiliar on-call engineer receives an alert at 2 AM. Without tribal knowledge, they use Cortex.io to quickly understand the system, identify the root cause, find the fix, resolve the incident, and initiate a post-mortem—all within minutes.

---

## Pre-Demo Setup

### Checklist
- [ ] All 5 services running in EKS (`kubectl get pods`)
- [ ] Frontend accessible via LoadBalancer URL
- [ ] Dynatrace showing all services healthy
- [ ] PagerDuty integration configured
- [ ] Cortex.io service catalog populated with all services
- [ ] Cortex.io scorecard configured for inventory-service
- [ ] Cortex.io Jira integration configured
- [ ] Browser tabs pre-opened:
  - Tab 1: Frontend application
  - Tab 2: Dynatrace (Services view)
  - Tab 3: PagerDuty (Incidents)
  - Tab 4: Cortex.io (Service Catalog)
  - Tab 5: Jira (Project board)

### Pre-Load Baseline (Optional)
Send a few orders through the UI before the demo so Dynatrace has baseline metrics:

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

### ACT 5: Cortex.io - The Hero Moment (10-12 minutes)

This is the star of the show. Take your time here.

---

#### 5A: Service Overview & Scorecard ⭐

**Action:** Click the Cortex.io link → lands on frontend service page

> "Immediately, our engineer sees everything they need to know about this service."

**Highlight:**
- Service description
- Owner team (Customer Experience)
- On-call contact (@cx-oncall)

**Action:** Point to the Service Scorecard

> "But look at this—the **Service Scorecard**. This gives us an instant health check of this service's operational readiness."

**Show Scorecard Items:**
- ✅ Has on-call configured
- ✅ Has documentation
- ✅ Has runbook linked
- ✅ Recent deployment (healthy)

> "The frontend looks well-maintained. All green. So the problem probably isn't here—let's trace the dependencies."

---

#### 5B: Visual Dependency Graph ⭐

**Action:** Navigate to the Dependencies view / Service Map in Cortex

> "Here's where Cortex really shines. Instead of guessing or digging through architecture docs, I can see the **visual dependency graph**."

**Show the dependency map:**
```
                    ┌─────────────────┐
                    │    frontend     │ ← Alert is here
                    └────────┬────────┘
                             │
                             ▼
                    ┌─────────────────┐
                    │  order-service  │
                    └────────┬────────┘
                             │
                             ▼
                    ┌─────────────────┐
                    │inventory-service│ ← Root cause is here
                    └────────┬────────┘
                             │
                             ▼
                    ┌─────────────────┐
                    │ payment-service │
                    └────────┬────────┘
                             │
                             ▼
                    ┌─────────────────┐
                    │notification-svc │
                    └─────────────────┘
```

> "I can see the entire request flow. The frontend depends on order-service, which depends on inventory-service, and so on."
>
> "Without Cortex, mapping this out might take 15-20 minutes of reading docs or asking around. Here, it's **instant and visual**."

**Action:** Click on **inventory-service** in the dependency graph

> "Let's check inventory-service..."

---

#### 5C: Scorecard Warning ⭐ KEY MOMENT

**Action:** On the inventory-service page, immediately highlight the Scorecard

> "Look at this! The scorecard for inventory-service is showing warnings."

**Show Scorecard Items:**
- ❌ **CPU limits below recommended threshold** ← THE WARNING
- ❌ No recent load testing
- ⚠️ Last deployment: 30+ days ago (potentially stale config)
- ✅ Has runbook linked
- ✅ Has on-call configured

> "Cortex was **already warning us** about this service. It flagged that CPU limits are below the recommended threshold."
>
> "This is proactive observability. If we'd been monitoring these scorecards, we might have caught this before it became an incident."

---

#### 5D: Recent Deployments & Changes ⭐

**Action:** Navigate to the Deployments or Recent Changes tab

> "Before I assume it's a configuration issue, let me check if anything changed recently."

**Show Recent Deployments:**
```
Last Deployments:
- 32 days ago: v1.2.0 - "Added inventory validation logic"
- 45 days ago: v1.1.0 - "Performance improvements"
- 60 days ago: v1.0.0 - "Initial release"
```

> "The last deployment was over a month ago. So this isn't a bad code deploy—this is a **scaling issue**. The service wasn't provisioned to handle this load."
>
> "This is valuable context. It tells me I don't need to rollback—I need to scale up resources."

---

#### 5E: Previous Incidents ⭐ KEY MOMENT

**Action:** Navigate to the Incidents or History tab

> "Let me check if this has happened before..."

**Show Previous Incidents:**
```
Previous Incidents:
- 3 months ago: "CPU Throttling - inventory-service" (Duration: 45 min)
- 6 months ago: "High latency during Black Friday" (Duration: 2 hours)
```

> "This is gold. Cortex shows that **this exact issue happened 3 months ago**. Same service, same symptoms—CPU throttling."
>
> "This tells me two things:"
> 1. "There's probably a runbook for this"
> 2. "We need a **permanent fix**, not just a band-aid"

> "Without Cortex, this tribal knowledge would be locked in someone's head or buried in a Slack thread from months ago."

---

#### 5F: Identifying the Root Cause & Owner

**Action:** On the inventory-service page, highlight the ownership info

> "Now I know what's wrong. But who do I talk to?"

**Show:**
- Owner Team: **Platform Infrastructure**
- Slack Channel: **#platform-eng**
- On-Call: **@platform-oncall**

> "Immediately, I see this is owned by a **different team** than the frontend. Without Cortex, I might have spent 20 minutes pinging the wrong people."

---

#### 5G: The Runbook ⭐ KEY MOMENT

**Action:** Click on the Runbooks section → Open the CPU Throttling runbook

> "And here it is—a **runbook** linked directly in Cortex: 'CPU Throttling Remediation.'"

**Walk through the runbook sections:**

1. **Symptoms:** "Response time degradation—check. ✅"
2. **Root Cause:** "CPU limits too low causing throttling under load. ✅"
3. **Diagnosis Steps:** "How to confirm in Dynatrace. ✅"
4. **Resolution:** "Here's the exact kubectl command to run. ✅"
5. **Verification:** "How to confirm the fix worked. ✅"

> "This is the power of Cortex. A brand new engineer, at 2 AM, with no prior knowledge of this system, has everything they need to resolve this incident."
>
> "No war rooms. No waking up senior engineers. No guessing."

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

> "We're increasing the CPU limit from 200 millicores to 1 full core, exactly as the runbook specified."

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

### ACT 8: Post-Incident - Jira Post-Mortem ⭐ (3 minutes)

This section showcases Cortex.io's workflow automation.

#### Talking Points:
> "The incident is resolved, but we're not done. Best practice says we need a post-mortem—especially since this is a **repeat incident**."
>
> "In most organizations, someone would have to manually create a Jira ticket, copy-paste all the details, and hope they don't miss anything."
>
> "Watch how Cortex handles this."

#### Action: Initiate Post-Mortem Workflow in Cortex

1. Navigate to the inventory-service page (or Workflows section)
2. Click **"Create Post-Mortem"** or **"Initiate Incident Review"**
3. Show the pre-populated form

> "Cortex has a post-mortem workflow built in. With one click, it creates a Jira ticket with all the context already filled in."

#### Action: Show the Pre-Populated Jira Ticket

**Jira Ticket Preview:**
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Title: [Post-Mortem] CPU Throttling - inventory-service - Dec 2024
Type: Post-Mortem
Priority: High
Assignee: Platform Infrastructure Team
Due Date: [5 business days from now]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

## Incident Summary
- **Service:** inventory-service
- **Duration:** ~15 minutes  
- **Severity:** High
- **Detected By:** Dynatrace
- **Resolved By:** [On-call engineer name]

## Impact
- **Services Affected:** frontend, order-service, inventory-service
- **User Impact:** Slow order processing, potential timeouts

## Timeline
- 14:00 - Alert triggered (Response time degradation on frontend)
- 14:05 - On-call engaged via PagerDuty
- 14:08 - Root cause identified via Cortex (CPU throttling)
- 14:12 - Fix applied (CPU increased to 1000m)
- 14:15 - Services recovered

## Root Cause
CPU limit (200m) insufficient for load. CPU-intensive inventory 
validation caused throttling under high traffic.

## Resolution
Applied kubectl patch to increase CPU limits to 1000m.

## Action Items
- [ ] Permanent fix: Update Helm chart / Terraform with correct CPU limits
- [ ] Add CPU utilization alert threshold to inventory-service
- [ ] Conduct load testing for inventory-service
- [ ] Review other services for similar resource constraints

## Related Links
- Runbook Used: [CPU Throttling Remediation]
- Cortex Service Page: [inventory-service]
- Dynatrace Problem: [link]
- Previous Similar Incident: [3 months ago]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

> "Look at this. Timeline, root cause, affected services, action items—all pre-populated from the incident context."
>
> "The on-call engineer doesn't have to remember what happened or dig through logs to write this up. Cortex captured it all."

#### Action: Create the Ticket

1. Click **"Create"** or **"Submit"**
2. Switch to Jira tab to show the ticket created

> "And now we have a Jira ticket for the post-mortem, assigned to the Platform Infrastructure team, due in 5 days."
>
> "This ensures follow-through. The permanent fix will happen."

---

### ACT 9: Update Service Catalog ⭐ (2 minutes)

#### Talking Points:
> "One more thing. We learned something important during this incident—inventory-service needs at least 500m CPU to operate under load."
>
> "Let's capture that knowledge in Cortex so the next engineer doesn't have to learn it the hard way."

#### Action: Edit Service Metadata in Cortex

1. Navigate to inventory-service page
2. Click **"Edit"** or **"Update Service"**
3. Add a note or custom field:

```yaml
# Example metadata update
x-cortex-notes: |
  ⚠️ IMPORTANT: This service requires minimum 500m CPU under load.
  Default 200m will cause throttling during traffic spikes.
  See runbook: cpu-throttling-remediation.md
  
x-cortex-resource-requirements:
  cpu-minimum: "500m"
  cpu-recommended: "1000m"
  memory-minimum: "512Mi"
```

> "Now this information is part of the service catalog. Anyone looking at this service in the future will see this warning."
>
> "This is how you turn tribal knowledge into institutional knowledge. It lives in Cortex, not in someone's head."

---

### ACT 10: The Wrap-Up (2 minutes)

#### Talking Points:

> "Let's recap what just happened:"
>
> "A new on-call engineer, with **no prior knowledge** of this system, was able to:"

| Step | Action | Time |
|------|--------|------|
| 1 | Received alert via PagerDuty | 0 min |
| 2 | Opened service in Cortex, saw scorecard warning | 1 min |
| 3 | Traced dependencies visually to find root cause | 2 min |
| 4 | Checked recent deployments (ruled out bad deploy) | 1 min |
| 5 | Found previous incidents (confirmed known issue) | 1 min |
| 6 | Located runbook with fix steps | 1 min |
| 7 | Applied fix from runbook | 2 min |
| 8 | Verified resolution | 2 min |
| 9 | Created post-mortem Jira ticket (automated) | 1 min |
| 10 | Updated service catalog with learnings | 1 min |
| | **Total Time** | **~12 min** |

> "Without Cortex, this same incident might have taken **45 minutes to an hour**:"
> - "15-20 minutes mapping dependencies and finding the right team"
> - "10-15 minutes tracking down documentation"
> - "10+ minutes coordinating across teams"
> - "Post-mortem often forgotten or delayed"

> "**Cortex.io reduced our MTTR by over 60%** and ensured proper follow-through with the automated post-mortem."

---

## Key Value Props Summary

| Challenge | Without Cortex | With Cortex |
|-----------|----------------|-------------|
| "Who owns this service?" | Slack search, asking around (15 min) | **Instant lookup** (5 sec) |
| "What does it depend on?" | Architecture docs, guessing (15 min) | **Visual dependency graph** (10 sec) |
| "Was there a recent deploy?" | CI/CD logs, Git history (10 min) | **Deployments tab** (10 sec) |
| "Has this happened before?" | Slack search, tribal knowledge (20 min) | **Previous incidents** (10 sec) |
| "Is there a runbook?" | Confluence search (10 min) | **Linked on service page** (5 sec) |
| "What about post-mortem?" | Manual Jira ticket, often forgotten | **Automated workflow** (1 min) |
| "How do we prevent this?" | Hope someone remembers | **Updated service catalog** (1 min) |

---

## Handling Questions

### "How does Cortex get this data?"
> "Teams maintain simple YAML files in their repos, or Cortex integrates with existing tools—GitHub, PagerDuty, Kubernetes, Jira, Dynatrace—to auto-discover and sync service information."

### "What if the runbook is out of date?"
> "Cortex integrates with Git, so runbooks are version-controlled and reviewed like code. You can also set up **scorecard rules** that flag services with stale documentation."

### "How did Cortex know about the previous incident?"
> "Cortex integrates with your incident management tools—PagerDuty, Opsgenie, etc.—and correlates incidents to services automatically."

### "Can we customize the post-mortem template?"
> "Absolutely. The Jira workflow is configurable. You can define what fields are auto-populated, what the template looks like, and even require certain action items."

### "What about the scorecard? How is that configured?"
> "Scorecards are rule-based. You define what 'good' looks like—has a runbook, CPU limits above threshold, recent security scan, etc.—and Cortex evaluates each service against those rules."

### "Does this work with our existing tools?"
> "Cortex integrates with 50+ tools: Dynatrace, Datadog, PagerDuty, Jira, GitHub, GitLab, Kubernetes, Terraform, and more. It's designed to be the single pane of glass that connects everything."

---

## Timing Summary

| Act | Content | Duration |
|-----|---------|----------|
| 1 | Set the Scene | 2 min |
| 2 | Trigger the Incident | 2 min |
| 3 | Dynatrace Detects | 2 min |
| 4 | PagerDuty Alert | 1 min |
| **5** | **Cortex.io Deep Dive** | **10-12 min** |
| | - Scorecard | 2 min |
| | - Dependency Graph | 2 min |
| | - Recent Deployments | 1 min |
| | - Previous Incidents | 2 min |
| | - Runbook | 2 min |
| 6 | Apply the Fix | 2 min |
| 7 | Verify Resolution | 2 min |
| **8** | **Post-Mortem Jira** | **3 min** |
| **9** | **Update Catalog** | **2 min** |
| 10 | Wrap-Up | 2 min |
| | **Total** | **23-25 min** |

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

## Cortex.io Feature Checklist for Demo

Ensure these are configured before the demo:

- [ ] **Service Catalog** - All 5 services registered
- [ ] **Scorecards** - Rules configured for CPU limits, runbooks, etc.
- [ ] **Dependencies** - Service relationships mapped
- [ ] **Deployments** - Integration with CI/CD or Git
- [ ] **Incidents** - Integration with PagerDuty
- [ ] **Runbooks** - Linked to inventory-service
- [ ] **Jira Workflow** - Post-mortem template configured
- [ ] **Owners** - Team assignments for all services

---

## Appendix: Demo Environment Details

| Component | URL/Details |
|-----------|-------------|
| Frontend | a64106ff620e641809e000cfa89f908e-280535655.us-east-2.elb.amazonaws.com |
| GitHub | https://github.com/CheemaKhasham/front-end, https://github.com/CheemaKhasham/order-service, https://github.com/CheemaKhasham/inventory-service, https://github.com/CheemaKhasham/payment-service, https://github.com/CheemaKhasham/notification-service |
| Cortex.io | https://app.cortex.io/catalog |
| Dynatrace | https://crr35672.apps.dynatrace.com/ |
| PagerDuty | Your org URL |
