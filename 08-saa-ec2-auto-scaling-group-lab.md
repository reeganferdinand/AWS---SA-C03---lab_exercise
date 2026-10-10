# SAA-C03 Lab 08: Amazon EC2 Auto Scaling – Create and Manage an Auto Scaling Group

**Repository filename:** `08-saa-ec2-auto-scaling-group-lab.md`

## 🎯 Aim

To create an EC2 Auto Scaling Group (ASG), distribute instances across multiple Availability Zones, integrate the ASG with an Application Load Balancer (ALB), and practise scaling out and scaling in by changing the desired capacity.

## 🧰 AWS Services Used

- Amazon EC2
- EC2 Launch Templates
- EC2 Auto Scaling Groups
- Application Load Balancer (ALB)
- Elastic Load Balancing Target Groups

## 🏗️ Architecture Diagram

```text
                 Users / Browser
                        |
                        v
                +----------------+
                |      ALB       |
                |  saa-alb-01    |
                +----------------+
                        |
                        v
                +----------------+
                | Target Group   |
                | saa-alb-tg-01  |
                +----------------+
                   /          \
                  v            v
          +------------+  +------------+
          |   EC2-01   |  |   EC2-02   |
          |  AZ-1      |  |  AZ-2      |
          +------------+  +------------+
                   ^            ^
                   |            |
              +---------------------+
              |   Auto Scaling      |
              |       Group         |
              +---------------------+
                         |
                  Launch Template
```

## 🧪 Procedure

### Step 1: Prepare the environment

1. Open **EC2 → Instances**.
2. Terminate the existing standalone EC2 instances if they are no longer needed.
3. Ensure you retain the ALB and target group from the previous lab.
4. Verify that the target group is `saa-alb-tg-01`.

> ⚠️ Terminating instances permanently deletes their instance-level data unless it has been backed up. Do not terminate instances used by other projects.

### Step 2: Create a Launch Template

1. Navigate to **EC2 → Launch Templates → Create launch template**.
2. Configure the template:

| Setting | Value |
|---|---|
| Name | `saa-asg-template-01` |
| Description | Launch template for SAA-C03 ASG lab |
| AMI | Amazon Linux 2, x86_64 |
| Instance type | `t2.micro`, if available and suitable |
| Key pair | Your existing tutorial key pair, if SSH access is needed |
| Security group | Existing web-server security group |
| Root volume | 8 GiB, gp2, as in the lecture |

3. Under **Advanced details → User data**, enter:

```bash
#!/bin/bash
yum install -y httpd
systemctl enable --now httpd
echo "Hello from $(hostname -f)" > /var/www/html/index.html
```

4. Create the launch template.
5. Confirm that the template is available for the next step.

**Important:** The EC2 security group must allow inbound HTTP traffic from the ALB security group. Use a suitable SSH rule only if required. Amazon Linux 2 may be unavailable or unsuitable for some newer environments; if so, use a supported Amazon Linux AMI and adapt the bootstrap commands.

### Step 3: Create the Auto Scaling Group

1. Navigate to **EC2 → Auto Scaling Groups → Create Auto Scaling group**.
2. Set the name to `saa-asg-01`.
3. Select launch template `saa-asg-template-01`, using version 1.
4. Choose the VPC used by your ALB.
5. Select subnets in multiple Availability Zones, preferably three if available.
6. Keep the Availability Zone distribution at **Balanced best effort**.

### Step 4: Attach the Load Balancer

1. Under load balancing, choose **Attach to an existing load balancer**.
2. Select the existing target group `saa-alb-tg-01`.
3. Enable both:
   - EC2 health checks
   - Elastic Load Balancing health checks
4. Continue to the group size configuration.

### Step 5: Configure Group Capacity

Set the initial values:

| Setting | Initial value |
|---|---:|
| Desired capacity | 1 |
| Minimum capacity | 1 |
| Maximum capacity | 1 |

Leave automatic scaling policies unconfigured for now. This lab focuses on manually changing the group's desired capacity.

5. Keep the remaining settings at their defaults unless required.
6. Review the configuration and create the Auto Scaling Group.

### Step 6: Verify Automatic Instance Creation

1. Open the ASG and check its **Activity** or **Activity history**.
2. Wait for the group to launch an EC2 instance.
3. Under **Instance management**, verify that one instance is running.
4. Open **EC2 → Target Groups → `saa-alb-tg-01` → Targets**.
5. Wait until the target's health status becomes **Healthy**.
6. Open the ALB DNS name in your browser.

Expected response:

```text
Hello from <EC2 hostname>
```

The instance may initially be unhealthy while Apache starts and the health checks run.

### Step 7: Scale Out from One to Two Instances

1. Open `saa-asg-01`.
2. Select **Edit** under group details.
3. Change the values:

| Setting | New value |
|---|---:|
| Desired capacity | 2 |
| Minimum capacity | 1 |
| Maximum capacity | 2 |

4. Save the changes.
5. Check **Activity history** for the new instance launch.
6. Wait for the second instance to become healthy in the target group.
7. Refresh the ALB DNS name several times.

Expected result: requests can be served by either EC2 instance.

### Step 8: Scale In from Two to One Instance

1. Edit `saa-asg-01`.
2. Change desired capacity from `2` to `1`.
3. Keep minimum capacity at `1` and maximum capacity at `2`.
4. Save the changes.
5. Check **Activity history** for an instance termination.
6. Verify that only one instance remains in the group.
7. Confirm that the remaining instance is healthy in the target group and the ALB still works.

## ✅ Verification Checklist

- [ ] Launch template created successfully.
- [ ] Auto Scaling Group created successfully.
- [ ] Initial desired capacity of one launched one instance.
- [ ] Instance registered automatically with the ALB target group.
- [ ] Target became healthy.
- [ ] Increasing desired capacity to two launched a second instance.
- [ ] Both instances became healthy.
- [ ] Reducing desired capacity to one terminated one instance.
- [ ] ALB continued serving requests.

## 📝 Key Exam Notes

- **Launch template:** Defines how new EC2 instances are launched.
- **Desired capacity:** The number of instances the ASG attempts to maintain.
- **Minimum capacity:** The lowest capacity the group is configured to maintain.
- **Maximum capacity:** The upper limit for the group's capacity.
- **Scale out:** Increase the number of instances.
- **Scale in:** Decrease the number of instances.
- **Health checks:** ASG can use EC2 health checks and, when enabled, load balancer health checks to identify unhealthy instances and replace them.
- **Multi-AZ deployment:** Helps distribute instances across Availability Zones for better availability.
- **Target group integration:** Instances launched by the ASG are automatically registered with the attached target group and deregistered when removed.

## 🧹 Cleanup

To avoid unnecessary AWS charges:

1. Set the ASG's desired capacity to `0` if you want to stop its EC2 instances.
2. Delete `saa-asg-01` when finished; verify that its instances are terminated.
3. Delete `saa-asg-template-01` if it is no longer needed.
4. Retain or delete the ALB, target group, and security groups according to whether they are needed for upcoming labs.

**Note:** Deleting the ASG does not necessarily delete the launch template or other shared resources. Check each resource separately.

## 🏁 Result

Successfully created an EC2 Auto Scaling Group using a launch template, integrated it with an Application Load Balancer, and verified scale-out and scale-in operations by adjusting desired capacity.
