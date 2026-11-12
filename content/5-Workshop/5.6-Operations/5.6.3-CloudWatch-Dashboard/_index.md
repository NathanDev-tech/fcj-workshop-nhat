---
title : "CloudWatch Dashboard"
date : 2024-01-01
weight : 3
chapter : false
pre : " <b> 5.6.3 </b> "
---

### 5.5.3. CloudWatch Dashboard

After completing this section, the system will meet the following requirements:

- The Dashboard is successfully displayed in CloudWatch.
- All Metrics are updated in real time.
- Alarms are configured and tested successfully.
- Alarm notifications via Email work properly.
- CloudWatch Log Insights queries have been prepared.

### Implementation Steps

#### Step 1. Create a CloudWatch Dashboard

1. Sign in to the **AWS Management Console**.
2. Navigate to the **CloudWatch** service.
3. Select **Dashboards**.
4. Click **Create dashboard**.

![Figure 66.](/images/5-Workshop/5.5-Raki-Operations/image066.png)

5. Enter a Dashboard name (for example: `raki-operational-dashboard`).

![Figure 67.](/images/5-Workshop/5.5-Raki-Operations/image067.png)

6. Select the option to create a new Dashboard.
7. If a Dashboard JSON template has already been prepared:
   - Select **Actions** → **View/Edit source**.
   - Paste the JSON template content.
   - Select **Save**.

![Figure 69.](/images/5-Workshop/5.5-Raki-Operations/image069.png)

---

#### Step 2. Add an ECS Metrics Widget

1. Open the Dashboard, select the newly created `raki-operational-dashboard`, and click **Add widget**.

![Figure 69.](/images/5-Workshop/5.5-Raki-Operations/image069.png)

2. Select Data type: **Metrics**, Preferred experience: **Metrics Console**.

3. Select Widget type: **Line**.

![Figure 68.](/images/5-Workshop/5.5-Raki-Operations/image068.png)

4. Click **Next**, then navigate to **ECS → ClusterName, ServiceName**.

5. Select the following **Metric Name** values:
   - CPU Utilization
   - Memory Utilization

![Figure 46.](/images/5-Workshop/5.5-Raki-Operations/image046.png)

6. Enter an appropriate Widget name and click **Create widget**.

![Figure 45.](/images/5-Workshop/5.5-Raki-Operations/image045.png)

---

#### Step 3. Add an Application Load Balancer (ALB) Metrics Widget

1. Open the Dashboard, select the newly created `raki-operational-dashboard`, and click **Add widget**.

![Figure 69.](/images/5-Workshop/5.5-Raki-Operations/image069.png)

2. Select Data type: **Metrics**, Preferred experience: **Metrics Console**.

3. Click **Add widget**.

4. Select **CloudWatch Metrics**, then click **Next**.

![Figure 70.](/images/5-Workshop/5.5-Raki-Operations/image070.png)

5. Select **Per AppELB, per AZ, per TG Metrics**, then add the following Metrics based on the Target Group configuration created in the previous stage:
   - Healthy Host Count
   - UnHealthy Host Count
   - Target Response Time
   - Request Count
   - HTTPCode_Target_5XX_Count

![Figure 60.](/images/5-Workshop/5.5-Raki-Operations/image060.png)
![Figure 110.](/images/5-Workshop/5.5-Raki-Operations/image110.png)
![Figure 111.](/images/5-Workshop/5.5-Raki-Operations/image111.png)
![Figure 112.](/images/5-Workshop/5.5-Raki-Operations/image112.png)

6. Enter an appropriate Widget name and click **Create widget**.

![Figure 113.](/images/5-Workshop/5.5-Raki-Operations/image113.png)

---

#### Step 4. Add an Amazon S3 Metrics Widget

1. Open the Dashboard, select the newly created `raki-operational-dashboard`, and click **Add widget**.

![Figure 69.](/images/5-Workshop/5.5-Raki-Operations/image069.png)

2. Select Data type: **Metrics**, Preferred experience: **Metrics Console**.

3. Click **Add widget**.

4. Select **CloudWatch Metrics**, then click **Next**.

![Figure 68.](/images/5-Workshop/5.5-Raki-Operations/image068.png)

5. In the **Browse** window, select the **S3** namespace, then select the Bucket to monitor.

![Figure 114.](/images/5-Workshop/5.5-Raki-Operations/image114.png)

4. Select the Storage Metrics of the `raki-frontend-dev` and `raki-logs` Buckets:
   - **BucketSizeBytes**
   - **NumberOfObjects**

![Figure 115.](/images/5-Workshop/5.5-Raki-Operations/image115.png)

5. Enter an appropriate Widget name and click **Create widget**.

![Figure 116.](/images/5-Workshop/5.5-Raki-Operations/image116.png)

---

#### Step 5. Add a CloudWatch Log Insights Widget

1. Open the Dashboard, select the newly created `raki-operational-dashboard`, and click **Add widget**.

![Figure 69.](/images/5-Workshop/5.5-Raki-Operations/image069.png)

2. Select **Log query**. Select the Log Groups of **ECS**, **Application**, and **ALB**.

![Figure 74.](/images/5-Workshop/5.5-Raki-Operations/image074.png)

4. Enter the following query in **CloudWatch Log Insights** to retrieve log records containing errors (`ERROR`, `Exception`, or status code `500`) from the last 7 days.

```sql
SOURCE "arn:aws:logs:ap-southeast-1:497172038341:log-group:/ecs/raki-backend" START=-604800s END=0s
| SOURCE "arn:aws:logs:ap-southeast-1:497172038341:log-group:/ecs/raki-task-be"
| fields @timestamp, @message
| filter @message like /ERROR|Exception|500/
| sort @timestamp desc
| limit 20