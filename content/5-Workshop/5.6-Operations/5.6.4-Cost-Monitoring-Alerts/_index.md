---
title : "Cost Monitoring & Alerts"
date : 2024-01-01
weight : 4
chapter : false
pre : " <b> 5.6.4 </b> "
---

### 5.5.4.1. Set Up an AWS Budget and Monthly Spending Limit

Open the **AWS Billing and Cost Management Console** → select **Budgets** and click **Create budget**, then choose the **Cost budget** type and configure the following settings:

**Budget name:** Enter `raki-monthly-budget`.

**Period:** Select **Monthly** to track the budget on a monthly cycle.

**Budget renewal type:** Select **Recurring budget** (the budget is automatically renewed on the first day of each month).

**Start month:** Select the month the budget should begin (for example: `Jul 2026`).

**Budgeting method:** Select **Fixed** to track based on a fixed monthly amount.

**Enter your budgeted amount ($):** Enter `40.00` (set a monthly budget limit of $40.00).

![image045.jpg](/fcj-workshop-nhat/images/5-Workshop/5.6-Operations/5.6.4-Cost-Monitoring-Alerts/image045.jpg)

![image046.jpg](/fcj-workshop-nhat/images/5-Workshop/5.6-Operations/5.6.4-Cost-Monitoring-Alerts/image046.jpg)

![image047.jpg](/fcj-workshop-nhat/images/5-Workshop/5.6-Operations/5.6.4-Cost-Monitoring-Alerts/image047.jpg)

![image048.jpg](/fcj-workshop-nhat/images/5-Workshop/5.6-Operations/5.6.4-Cost-Monitoring-Alerts/image049.jpg)

### 5.5.4.2. Configure the Budget Scope and Tags

**Budget scope:** Select **Filter specific AWS cost dimensions** to filter costs for this project instead of monitoring the entire AWS account.

**Filters:** Filter by **Tag: Project included (1)** with the value `RakiBookery`.

**Advanced options:** Keep **Aggregate costs by** set to **Unblended costs**.

**Tags (optional):** Add resource-management tags to the Budget:

- **Key:** `Project` | **Value - optional:** `RakiBookery`
- **Key:** `ManagedBy` | **Value - optional:** `CloudFormation`

![image046.jpg](/images/5-Workshop/5.5-Raki-Operations/image046.jpg)

### 5.5.4.3. Configure Alerts and Review Budget Details

Open **Budgets** → select `raki-monthly-budget` to view the details and check **Budget health** as well as configure the alert thresholds (**Alerts**):

**Budget health:** The status displays as **Healthy** when the current cost (**Current vs. budgeted**) has not exceeded the $40.00 limit.

**Configure 3 alert thresholds (Alerts):**

- **Actual cost > 50%:** Triggers an alert when actual spending exceeds **50%** ($20.00) of the $40.00 budget.
- **Actual cost > 70%:** Triggers an alert when actual spending exceeds **70%** ($28.00) of the $40.00 budget.
- **Forecasted cost > 90%:** Triggers a forecast alert when the estimated end-of-month cost exceeds **90%** ($36.00) of the $40.00 budget.

![image047.jpg](/images/5-Workshop/5.5-Raki-Operations/image047.jpg)
