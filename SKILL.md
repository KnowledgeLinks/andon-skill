---
description: Assist the user in responding to Andon alerts (process anomalies, SLA violations, or metric failures) by stopping the line, investigating root causes, and implementing countermeasures.
---

# Andon Skill (Stop-the-Line Protocol)

In the Toyota Production System, *andon* is the practice of empowering any front-line
worker to signal — and if necessary halt — a production line when a fault is observed.
The key principle: **quality problems surface immediately rather than propagating
downstream.** Management's trust in front-line judgment is what makes andon work.

In a lean library, andon applies to any repetitive workflow — metadata batch loading,
cataloging pipelines, patron-facing services, or automated BML loop tasks. An andon
pull may come from a staff member, an automated monitoring agent, or a metric
threshold breach. The goal is to stop, investigate, fix, and resume — in that order.

This skill helps teams and AI agents halt development or operations when critical
metrics fail or process anomalies are detected, then implement permanent countermeasures
before resuming normal speed.

## Step 1: Triaging the Andon Pull

Ask the user or inspect the logs/metrics to determine:
1. **What triggered the alert?** (e.g., API latency > 2000ms, checkout failure rate > 5%, cataloging backlog > 48h, MARC encoding errors in a batch load, critical patron complaint, LLM token cost spike on a metadata enhancement task).
2. **What is the immediate impact?** Who is affected (patrons, staff, specific departments)? Is a patron-initiated workflow blocked?
3. **Is the workflow type suited to a full stop?** Andon works best for short-cycle, highly repetitive workflows where help is available to respond immediately. For long-running or complex workflows, a partial stop (halt the affected branch only) may be more appropriate.
4. **Is the line officially "stopped"?** Confirm that no new features or non-critical commits will be deployed until the andon is cleared. For automated pipelines, confirm the DAG run is paused or failed tasks are not retried.

---

## Step 2: The Five Whys (Root Cause Analysis)

Guide the team through a **Five Whys** analysis to move past superficial symptoms.
*For each level, ask "Why did this happen?" based on observed system facts, not assumptions.*

1. **Why** did the symptom occur? (e.g., The batch load script failed.)
2. **Why** did that happen? (e.g., An invalid Unicode encoding in an author name field caused a parse error.)
3. **Why** did the invalid encoding enter the batch? (e.g., The vendor MARC records were not normalized before ingestion.)
4. **Why** wasn't normalization applied? (e.g., The normalization step was skipped to speed up the MVP in Loop 1.)
5. **Why** was it skipped? (e.g., No quality threshold was defined in the Measure step, so the risk was not visible.)

Stop at fewer than five whys if a genuine root cause is found earlier. The goal is
reaching the systemic cause, not hitting the number five.

---

## Step 3: Actionable Countermeasures

Develop two types of solutions:

1. **Short-term containment**: What is the minimum action needed to safely resume
   operations without propagating the fault downstream?  
   (e.g., Remove the malformed records from the batch, rerun with clean input; pause
   the Airflow DAG run; roll back a deployment.)

2. **Long-term prevention**: What changes must be made to the Build or Measure tasks
   in the next BML loop to prevent recurrence?  
   (e.g., Add a pre-ingestion validation task to the DAG; define a record quality
   threshold as an actionable metric; implement LLM-based monitoring agents to alert
   staff when task duration exceeds a threshold.)

Remind the user: the long-term countermeasure belongs in the **next BML loop's Build
task list** — this is the direct link between andon and the BML DAG.

---

## Step 4: Resolution & Retrospective

Organizational memory is critical — an andon pull that is fixed but not documented
will recur. Produce an andon incident record for the team's shared knowledge base:

```
Andon Pull Record
-----------------
Trigger: <detailed symptom and metric threshold breached>
Workflow affected: <pipeline, service, or workflow name>
Impact: <who was affected (patrons / staff / departments), duration>
Patron-initiated workflow blocked: [YES / NO]

Root Cause (5 Whys):
  1. <Why 1>
  2. <Why 2>
  3. <Why 3>
  4. <Why 4>
  5. <Why 5 / root cause>

Countermeasures:
  - Containment: <short-term fix applied>
  - Prevention (BML Loop task): <change to add to next Build or Measure task list>

Status: [CLEARED / PENDING]
Diffusion: <who else in the library should know about this — other teams, administration>
```

Ask the user if they want to save this record to an incident log or issue tracker,
and whether the prevention countermeasure should be added to an open BML loop's
Build task list now.
