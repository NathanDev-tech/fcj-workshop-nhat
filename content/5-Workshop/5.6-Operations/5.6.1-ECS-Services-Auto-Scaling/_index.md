---
title : "ECS Services + Auto-Scaling"
date : 2024-01-01
weight : 1
chapter : false
pre : " <b> 5.6.1 </b> "
---


### 5.6.1.1. Create the Backend Service

1. Open **ECS → Clusters → raki-prod-cluster → Create service**.
2. Select the `raki-task-be` task definition family, set the service name to `rakibookery-prod-service`, and select the latest **Task definition revision**.

![picCreateECS12](/fcj-workshop-nhat/images/5-Workshop/5.6-Operations/5.6.1-ECS-Services-Auto-Scaling/081422.png)

3. In the networking section, select the application VPC, its **private subnets**, and the ECS task security group.

![image040](/fcj-workshop-nhat/images/5-Workshop/5.6-Operations/5.6.1-ECS-Services-Auto-Scaling/081501.png)

4. Under **Load balancing**, select the `raki-prod-egde-alb` **Application Load Balancer**, the existing `80:HTTP` listener, container port `8000`, and the existing `raki-prod-php-tg` target group.

![image041](/fcj-workshop-nhat/images/5-Workshop/5.6-Operations/5.6.1-ECS-Services-Auto-Scaling/081752.png)

5. Review the configuration and select **Create**. ECS creates the task, registers its IP address with the frontend target group, and registers it with Cloud Map.

![image042](/fcj-workshop-nhat/images/5-Workshop/5.6-Operations/5.6.1-ECS-Services-Auto-Scaling/081902.png)

### 5.6.1.2. Enable Auto Scaling and Create a CPU Policy

1. Open the `rakibookery-prod-service` service, go to the **Service auto scaling** tab, and select **Update**.
2. Enable **Use service auto scaling**. Under *Capacity limits*, set the minimum number of tasks to `2` and the maximum to `6`. This keeps two backend tasks ready to serve requests while limiting scale-out to six tasks to control costs.

![image003](/fcj-workshop-nhat/images/5-Workshop/5.6-Operations/5.6.1-ECS-Services-Auto-Scaling/082119.png)

3. In the scaling policy section, choose **Target tracking**. Name the policy `cpu-70-target-tracking` and select the **ECSServiceAverageCPUUtilization** metric. This metric measures the average CPU usage across all running tasks in the service.
4. Set the *Target value* to `70`. When average CPU usage exceeds 70%, ECS Service Auto Scaling adds tasks to distribute the load. When CPU usage falls, ECS can remove tasks but never below the minimum of two.
5. Set *Scale-out cooldown* to `60 seconds` and *Scale-in cooldown* to `300 seconds`, then save the policy. After each scale-out event, ECS waits 60 seconds for the new task to start and register with the target group. Scale-in waits five minutes to prevent fluctuating load from repeatedly adding and removing tasks.

![image001](/fcj-workshop-nhat/images/5-Workshop/5.6-Operations/5.6.1-ECS-Services-Auto-Scaling/082418.png)

After saving, the `cpu-70-target-tracking` policy monitors the service CPU within the range of `2` to `6` tasks.

![image009](/fcj-workshop-nhat/images/5-Workshop/5.6-Operations/5.6.1-ECS-Services-Auto-Scaling/082915.png)

![image007](/fcj-workshop-nhat/images/5-Workshop/5.6-Operations/5.6.1-ECS-Services-Auto-Scaling/083047.png)
