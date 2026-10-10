# SAA-C03 Lab 09: Amazon EC2 Auto Scaling – Configure Automatic Scaling Policies

**Repository filename:** `09-saa-ec2-auto-scaling-policies-lab.md`

## 🎯 Aim

To configure and test automatic scaling policies for an EC2 Auto Scaling Group (ASG), using target tracking based on CPU utilization, and understand dynamic scaling, predictive scaling, and scheduled actions.

## 🧰 AWS Services Used

- Amazon EC2
- EC2 Auto Scaling Groups
- Amazon CloudWatch
- Application Load Balancer (ALB)

## 🏗️ Architecture Diagram

```text
              EC2 Instance Metrics
                 CPU Utilization
                        |
                        v
                +----------------+
                |  CloudWatch    |
                |    Alarms      |
                +----------------+
                        |
                        v
                +----------------+
                | Target Tracking|
                | Scaling Policy |
                +----------------+
                        |
                        v
                +----------------+
                |      ASG       |
                | Min: 1, Max: 3 |
                +----------------+
                  /     |      \
                 v      v       v
              EC2-01  EC2-02   EC2-03
                        |
                        v
                  ALB Target Group
```

## 📚 Types of Automatic Scaling

| Type | Purpose |
|---|---|
| Dynamic scaling | Adjust capacity based on current demand and metrics |
| Predictive scaling | Forecast future demand using historical patterns |
| Scheduled actions | Change capacity at predefined times |

Dynamic scaling policy types:

- **Target tracking:** Maintain a metric near a target value.
- **Step scaling:** Adjust capacity by different amounts based on alarm severity.
- **Simple scaling:** Perform a scaling adjustment when a CloudWatch alarm triggers.

## 🧪 Procedure

### Step 1: Verify the Auto Scaling Group

1. Open **EC2 → Auto Scaling Groups**.
2. Select `saa-asg-01` from Lab 08.
3. Verify that the group has one running instance.
4. Confirm that the launch template and ALB target group are configured correctly.

### Step 2: Configure Group Capacity

1. Select the ASG and choose **Edit**.
2. Set the following values:

| Setting | Value |
|---|---:|
| Desired capacity | 1 |
| Minimum capacity | 1 |
| Maximum capacity | 3 |

3. Save the configuration.

This allows the ASG to scale between one and three instances.

### Step 3: Create a Target Tracking Policy

1. Open the ASG's **Automatic scaling** tab.
2. Choose **Create dynamic scaling policy**.
3. Configure the policy:

| Setting | Value |
|---|---|
| Policy name | `saa-asg-cpu-target-01` |
| Policy type | Target tracking |
| Metric type | Average CPU utilization |
| Target value | 40% |

4. Create the policy.

AWS automatically creates CloudWatch alarms to manage scale-out and scale-in decisions.

### Step 4: Generate CPU Load

Connect to the running EC2 instance using EC2 Instance Connect or SSH.

For Amazon Linux 2, install the `stress` utility using an appropriate package source for your environment. If the package is available through the configured repositories, run:

```bash
sudo amazon-linux-extras install epel -y
sudo yum install stress -y
```

Then generate CPU load:

```bash
stress -c 4
```

This starts four CPU worker processes. On an instance with fewer than four vCPUs, CPU utilization may rise substantially.

**Note:** The installation commands may not work on every AMI or repository configuration. Use a compatible installation method if necessary.

### Step 5: Monitor CPU Utilization and Scaling

1. Open **EC2 → Instances** and select the instance.
2. Open its **Monitoring** tab and observe CPU utilization.
3. Open the ASG and select **Activity history**.
4. Wait for CloudWatch to collect sufficient metrics and evaluate the alarm.
5. Observe whether the ASG launches additional instances.
6. Under **Instance management**, verify the current instance count.

The ASG may scale out when CPU utilization remains above the target for the policy's evaluation period. Scaling is not necessarily immediate.

### Step 6: Inspect CloudWatch Alarms

1. Open **CloudWatch → Alarms**.
2. Locate the alarms created by the target tracking policy.
3. Inspect the alarm state, metric, threshold, and evaluation periods.
4. Observe how the alarms support scale-out and scale-in decisions.

The exact alarm names and thresholds are managed by AWS and may vary. Target tracking uses separate scale-out and scale-in behavior to maintain the configured target.

### Step 7: Observe Scale-In

1. Stop the CPU stress process by pressing `Ctrl+C` in the terminal where it is running.
2. If necessary, verify that no stress processes remain:

```bash
pgrep -a stress
```

3. Return to CloudWatch and monitor CPU utilization.
4. Open the ASG's **Activity history**.
5. Wait for utilization to fall and the scaling policy to initiate scale-in.
6. Verify that the instance count decreases toward the desired capacity.

Scale-in may take several minutes or longer, depending on the metrics and policy evaluation periods.

## 🔍 Additional Concepts to Explore

### A. Scheduled Scaling

1. Open the ASG's **Automatic scaling** tab.
2. Create a scheduled action.
3. Specify the desired, minimum, or maximum capacity to apply.
4. Configure a one-time or recurring schedule.
5. Review the start and end times before saving.

Use this when demand patterns are predictable, such as a planned sale or a recurring busy period.

### B. Predictive Scaling

Explore the predictive scaling policy options in the ASG console.

Review the historical metrics, forecast configuration, and target utilization settings. Predictive scaling uses historical demand patterns to forecast future capacity needs.

A sufficiently useful forecast requires historical metric data; this behavior may not be demonstrable immediately in a newly created lab.

### C. Step Scaling and Simple Scaling

Review the other dynamic scaling options:

- **Step scaling:** Configure different capacity adjustments for different alarm breach ranges.
- **Simple scaling:** Configure a capacity adjustment triggered by a CloudWatch alarm.

These options are useful to understand for the SAA-C03 exam even if you do not create separate policies for this lab.

## ✅ Verification Checklist

- [ ] ASG minimum capacity is 1.
- [ ] ASG maximum capacity is 3.
- [ ] Target tracking policy created with a CPU target of 40%.
- [ ] CPU load generated on an EC2 instance.
- [ ] CloudWatch alarms inspected.
- [ ] Scale-out activity observed, if the load and evaluation period trigger it.
- [ ] CPU load stopped.
- [ ] Scale-in activity observed after CPU utilization decreases.
- [ ] ASG returns toward the required capacity.

## 📝 Key Exam Notes

- **Target tracking:** Keeps a selected metric near a specified target.
- **Step scaling:** Applies different adjustments according to alarm breach ranges.
- **Simple scaling:** Performs a scaling adjustment after an alarm triggers, with a cooldown behavior.
- **Predictive scaling:** Uses historical patterns to forecast demand.
- **Scheduled scaling:** Changes capacity at specified times.
- **CloudWatch:** Supplies metrics and alarms used by scaling policies.
- **Minimum, desired, and maximum capacity:** Control the lower bound, intended capacity, and upper bound of the ASG.

## 🧹 Cleanup

1. Stop all CPU stress processes.
2. Delete `saa-asg-cpu-target-01` when the experiment is complete.
3. Set the ASG's desired capacity to 1 if you want to retain the Lab 08 environment.
4. If you are finished with the entire ASG lab series, delete the ASG and verify that its instances are terminated.
5. Remove any temporary resources created solely for this experiment.

Do not manually delete CloudWatch alarms while the target tracking policy still manages them; remove the policy first.

## 🏁 Result

Successfully explored automatic scaling strategies and configured a target tracking policy based on average CPU utilization. Observed how CloudWatch alarms and ASG activity history help explain scale-out and scale-in operations.
