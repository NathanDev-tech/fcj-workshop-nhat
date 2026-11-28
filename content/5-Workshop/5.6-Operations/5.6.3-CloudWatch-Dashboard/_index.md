---
title : "CloudWatch Dashboard"
date : 2024-01-01
weight : 3
chapter : false
pre : " <b> 5.6.3 </b> "
---

After completing this section, the system will meet the following requirements:

- Dashboard is displayed in CloudWatch
- All Metrics are updated in real time
- Alarms are configured and tested
- Alarm notifications via Email work properly
- CloudWatch Log Insights queries have been prepared

### 5.5.3.1. Implementation Steps
#### Step 1. Create a CloudWatch Dashboard

1. Sign in to the **AWS Management Console**.
2. Navigate to the **CloudWatch** service.
3. Select **Dashboards**.
4. Click **Create dashboard**.

![Figure 66.](/fcj-workshop-nhat/images/5-Workshop/5.6-Operations/5.6.3-CloudWatch-Dashboard/image066.png)

5. Enter a Dashboard name (for example: `raki-operational-dashboard`).

![Figure 67.](/fcj-workshop-nhat/images/5-Workshop/5.6-Operations/5.6.3-CloudWatch-Dashboard/image067.png)

6. Select the option to create a new Dashboard.
7. If a Dashboard JSON template has already been prepared:
   - Select **Actions** → **View/Edit source**.
   - Paste the JSON template content.
   - Select **Save**.
<!-- 
![Figure 69.](/images/5-Workshop/5.5-Raki-Operations/image069.png) -->

---

#### Step 2. Add an ECS Metrics Widget

1. Open the Dashboard, select the newly created `raki-operational-dashboard`, and click **Add widget**.

![Figure 69.](/fcj-workshop-nhat/images/5-Workshop/5.6-Operations/5.6.3-CloudWatch-Dashboard/image069.png)

2. Select Data type: **Metrics**, Preferred experience: **Metrics Console**

3. Select Widget type: **Line**
![Figure 69.](/fcj-workshop-nhat/images/5-Workshop/5.6-Operations/5.6.3-CloudWatch-Dashboard/image069.png)

4. Click **Next**, then navigate to **ECS → ClusterName, ServiceName**

5. Select the following **Metric Name** values:
   - CPU Utilization
   - Memory Utilization

![Figure 70.](/fcj-workshop-nhat/images/5-Workshop/5.6-Operations/5.6.3-CloudWatch-Dashboard/image070.png)

6. Enter an appropriate Widget name and click **Create widget**.

![Figure 76.](/fcj-workshop-nhat/images/5-Workshop/5.6-Operations/5.6.3-CloudWatch-Dashboard/image076.png)

---

#### Step 3. Add an Application Load Balancer (ALB) Metrics Widget

1. Open the Dashboard, select the newly created `raki-operational-dashboard`, and click **Add widget**.

![Figure 71.](/fcj-workshop-nhat/images/5-Workshop/5.6-Operations/5.6.3-CloudWatch-Dashboard/image071.png)

2. Select Data type: **Metrics**, Preferred experience: **Metrics Console**

3. Click **Add widget**.

4. Select **CloudWatch Metrics**, then click **Next**.

![Figure 72.](/fcj-workshop-nhat/images/5-Workshop/5.6-Operations/5.6.3-CloudWatch-Dashboard/image072.png)

5. Select **Per AppELB, per AZ, per TG Metrics**, then add the following Metrics based on the Target Group configuration from the previous stage:
   - Healthy Host Count
   - UnHealthy Host Count
   - Target Response Time
   - Request Count
   - HTTPCode_Target_5XX_Count

![Figure 72.](/fcj-workshop-nhat/images/5-Workshop/5.6-Operations/5.6.3-CloudWatch-Dashboard/image072.png)

6. Enter an appropriate Widget name and click **Create widget**.

![Figure 74.](/fcj-workshop-nhat/images/5-Workshop/5.6-Operations/5.6.3-CloudWatch-Dashboard/image074.png)

---

#### Step 4. Add an Amazon S3 Metrics Widget

1. Open the Dashboard, select the newly created `raki-operational-dashboard`, and click **Add widget**.

[Figure 71.](/fcj-workshop-nhat/images/5-Workshop/5.6-Operations/5.6.3-CloudWatch-Dashboard/image071.png)

2. Select Data type: **Metrics**, Preferred experience: **Metrics Console**

3. Click **Add widget**.

4. Select **CloudWatch Metrics**, then click **Next**.

[Figure 68.](/fcj-workshop-nhat/images/5-Workshop/5.6-Operations/5.6.3-CloudWatch-Dashboard/image068.png)

5. In the **Browse** window, select the **S3** namespace, then select the Bucket to monitor.

[Figure 73.](/fcj-workshop-nhat/images/5-Workshop/5.6-Operations/5.6.3-CloudWatch-Dashboard/image073.png)

4. Select the Storage Metrics of the `raki-frontend-dev` and `raki-logs` Buckets:
   - **BucketSizeBytes**
   - **NumberOfObjects**

[Figure 73.](/fcj-workshop-nhat/images/5-Workshop/5.6-Operations/5.6.3-CloudWatch-Dashboard/image073.png)

5. Enter an appropriate Widget name and click **Create widget**.

[Figure 75.](/fcj-workshop-nhat/images/5-Workshop/5.6-Operations/5.6.3-CloudWatch-Dashboard/image075.png)

---

<!-- #### Step 5. Add a CloudWatch Log Insights Widget
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
```

![Figure 75.](/images/5-Workshop/5.5-Raki-Operations/image075.png)

5. Review the returned results, save the Widget to the Dashboard by clicking **create**, then click **Save** on the `Dashboardsraki-operational-dashboard` screen to save all Widgets.

![Figure 76.](/images/5-Workshop/5.5-Raki-Operations/image076.png)

---

#### Step 6. Add an Amazon RDS Metrics Widget

Repeat the same steps above as follows:

1. Click **Add widget**.
2. Select Metrics from **Amazon RDS**.
3. Select the Database Instance.
4. Add the following Metrics:
   - CPU Utilization
   - Database Connections
   - Read Latency
   - Write Latency
   - Free Storage Space
5. Save the Widget.

---

#### Step 7. Create CloudWatch Alarms

CloudWatch Alarms monitor system Metrics and automatically detect when resources exceed the allowed threshold. In this section, create an alarm to monitor ECS Service CPU utilization.

1. Navigate to **CloudWatch** → **Alarms**.

2. Click **Create alarm** to create a new alarm.

![alt text](image-3.png)

3. At the **Specify metric and conditions** step, click **Select metric** to choose the Metric to monitor.

![alt text](image-4.png)

4. In the Metric list, select:

   - **ECS**
   - **ClusterName, ServiceName**

![alt text](image-5.png)

5. Select the **CPUUtilization** Metric of the ECS Service, then click **Select metric**.

![alt text](image-6.png)

6. Configure the alarm trigger condition:

   - **Statistic:** Average
   - **Period:** 5 minutes
   - **Threshold type:** Static
   - **Whenever CPUUtilization is:** Greater than
   - **Threshold value:** 80

CloudWatch will switch the Alarm to the **ALARM** state when the average CPU utilization exceeds **80%** during the evaluation period.

![alt text](image-7.png)
![alt text](image-8.png)

7. At the **Configure actions** step, choose the action to take when the Alarm is triggered. You can send a notification via an **SNS Topic** or skip this step if you only need to monitor the status.

![Figure 57.](/images/5-Workshop/5.5-Raki-Operations/image057.png)

8. Enter an Alarm name, for example:

   - **Alarm name:** `ECS-Backend-High-CPU-Alarm`

You can also add a description for easier management later.

![Figure 55.](/images/5-Workshop/5.5-Raki-Operations/image055.png)

9. Review all settings and click **Create alarm** to complete.

![Figure 64.](/images/5-Workshop/5.5-Raki-Operations/image064.png)

10. After the alarm is successfully created, it will appear in the **CloudWatch Alarms** list with an initial status of **OK**. When the CPU value exceeds the configured threshold, the status will automatically change to **ALARM**.

![Figure 65.](/images/5-Workshop/5.5-Raki-Operations/image065.png)

> **Note:** Similarly, you can create additional CloudWatch Alarms to monitor the system, such as:
>
> - **HTTPCode_Target_5XX_Count** > 10 errors/minute.
> - **MemoryUtilization** > 80%.
> - **TargetResponseTime** exceeds the desired threshold.
> - **HealthyHostCount** drops below the minimum required count.

#### Step 8. Create an SNS Topic for Notifications and Register an Email Subscription

1. Navigate to the **Amazon SNS** service, select **Topics**, and click **Create topic**.

2. Select **Standard** type.

3. Enter a Topic name (for example: `raki-Alerts-Topic`).

4. Complete the Topic creation.

![alt text](image-2.png)

![Figure 47.](/images/5-Workshop/5.5-Raki-Operations/image047.png)

![alt text](image-1.png)

5. Open the newly created Topic and select **Create subscription**.

![Figure 48.](/images/5-Workshop/5.5-Raki-Operations/image048.png)

6. Protocol: select **Email**.

7. Enter the operations team email address or a personal email address.

8. Submit the Subscription, open the email, and click **Confirm Subscription** to activate it.

![Figure 49.](/images/5-Workshop/5.5-Raki-Operations/image049.png)

--- -->
