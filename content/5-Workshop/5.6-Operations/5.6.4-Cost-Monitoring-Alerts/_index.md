---
title : "Cost Monitoring & Alerts"
date : 2024-01-01
weight : 4
chapter : false
pre : " <b> 5.6.4 </b> "
---

### 5.5.4.1. Set Up an AWS Budget and Monthly Spending Limit

Open the **AWS Billing and Cost Management console**, select **Budgets**, and choose **Create budget**. Select **Cost budget**, then configure the following settings:

**Budget name:** Enter `raki-monthly-budget`.

**Period:** Select **Monthly** to track the budget each month.

**Budget renewal type:** Select **Recurring budget** so the budget is automatically renewed on the first day of each month.

**Start month:** Select the month when the budget should begin, for example `Jul 2026`.

**Budgeting method:** Select **Fixed** to track a fixed monthly spending amount.

**Enter your budgeted amount ($):** Enter `40.00` to set a monthly budget limit of $40.00.

![image045.jpg](/images/5-Workshop/5.5-Raki-Operations/image045.jpg)

### 5.5.4.2. Configure the Budget Scope and Tags

**Budget scope:** Select **Filter specific AWS cost dimensions** to track costs for this project instead of the entire AWS account.

**Filters:** Filter by **Tag: Project included (1)** with the value `RakiBookery`.

**Advanced options:** Keep **Aggregate costs by** set to **Unblended costs**.

**Tags (optional):** Add resource-management tags to the budget:

- **Key:** `Project` | **Value – optional:** `RakiBookery`
- **Key:** `ManagedBy` | **Value – optional:** `CloudFormation`

![image046.jpg](/images/5-Workshop/5.5-Raki-Operations/image046.jpg)

### 5.5.4.3. Configure Alerts and Review Budget Details

Open **Budgets**, select `raki-monthly-budget`, and review the budget details, including **Budget health** and its alert thresholds:

**Budget health:** The status is **Healthy** when the current cost (**Current vs. budgeted**) has not exceeded the $40.00 limit.

**Configure these three alert thresholds:**

- **Actual cost > 50%:** Sends an alert when actual spending exceeds **50%** ($20.00) of the $40.00 budget.
- **Actual cost > 70%:** Sends an alert when actual spending exceeds **70%** ($28.00) of the $40.00 budget.
- **Forecasted cost > 90%:** Sends an alert when forecasted end-of-month spending exceeds **90%** ($36.00) of the $40.00 budget.

![image047.jpg](/images/5-Workshop/5.5-Raki-Operations/image047.jpg)