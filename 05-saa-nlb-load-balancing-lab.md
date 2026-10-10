# SAA-C03 Lab 05: Network Load Balancer – TCP Traffic Distribution Across EC2 Instances

## 🎯 Tasks
1. Create an internet-facing **Network Load Balancer (NLB)**
2. Configure an NLB security group
3. Create a TCP target group and register two EC2 instances
4. Configure HTTP health checks
5. Update the EC2 security group to allow traffic from the NLB
6. Verify traffic distribution and clean up resources

---

## 🧩 Step 1: Create NLB Security Group

1. Go to **EC2 → Security Groups → Create security group**.
2. Configure:

- **Name:** `saa-nlb-sg-01`
- **VPC:** Same VPC as the EC2 instances

3. Add an inbound rule:

- **Type:** HTTP
- **Port:** 80
- **Source:** `0.0.0.0/0`

4. Create the security group.

---

## 🔹 Step 2: Create a Target Group

1. Go to **EC2 → Target Groups → Create target group**.
2. Configure:

- **Target type:** Instances
- **Name:** `saa-nlb-tg-01`
- **Protocol:** TCP
- **Port:** 80
- **VPC:** Same VPC as EC2
- **Health check protocol:** HTTP
- **Health check path:** `/`
- **Healthy threshold:** `2`
- **Health check interval:** `5 seconds`
- **Health check timeout:** `2 seconds`

3. Click **Next**.
4. Register `saa-alb-ec2-01` and `saa-alb-ec2-02` on port 80.
5. Create the target group.

👉 Reuse the EC2 instances from Lab 03 if they are still available.

---

## 🔹 Step 3: Create a Network Load Balancer

1. Go to **EC2 → Load Balancers → Create load balancer**.
2. Select **Network Load Balancer**.
3. Configure:

- **Name:** `saa-nlb-01`
- **Scheme:** Internet-facing
- **IP address type:** IPv4
- **VPC:** Same VPC as EC2
- **Network mappings:** Select subnets in at least two Availability Zones
- **Security group:** `saa-nlb-sg-01`

4. Under **Listeners and routing**, configure:

- **Protocol:** TCP
- **Port:** 80
- **Default action:** Forward to `saa-nlb-tg-01`

5. Create the load balancer.
6. Wait until its state becomes **Active**.

👉 Each enabled Availability Zone receives an NLB address. You can use an Elastic IP where supported and configured.

---

## 🔹 Step 4: Update the EC2 Security Group

1. Go to **EC2 → Security Groups**.
2. Select `saa-alb-ec2-sg-01`, attached to the EC2 instances.
3. Choose **Edit inbound rules**.
4. Add a rule:

- **Type:** HTTP
- **Port:** 80
- **Source:** Security group → `saa-nlb-sg-01`

5. Save the rules.

👉 The EC2 instances must allow traffic from the NLB security group for the health checks and requests to succeed in this lab setup.

---

## 🔹 Step 5: Verify Target Health

1. Go to **EC2 → Target Groups → `saa-nlb-tg-01` → Targets**.
2. Wait until both EC2 instances show **Healthy**.
3. If targets are unhealthy, verify:

- EC2 instances are running.
- The web server is listening on port 80.
- The health check path `/` responds successfully.
- Security group rules allow the required traffic.

---

## 🔹 Step 6: Test Load Balancing

1. Open **EC2 → Load Balancers → `saa-nlb-01`**.
2. Copy its DNS name.
3. Open the following URL in a browser:

```text
http://<NLB-DNS-NAME>
```

4. Refresh the page several times.
5. Verify that responses can come from both EC2 instances.
6. Confirm both targets remain **Healthy**.

---

## 🧹 Step 7: Cleanup

1. Delete `saa-nlb-01`.
2. Delete `saa-nlb-tg-01`.
3. Delete `saa-nlb-sg-01` after confirming it is no longer in use.

If you are also finishing Labs 03 and 04:

4. Terminate `saa-alb-ec2-01` and `saa-alb-ec2-02`.
5. Delete the ALB and target group.
6. Delete unused security groups.

⚠️ The NLB can incur charges while provisioned. Clean up promptly after testing.

---

## ✅ Verification

- [ ] NLB created and Active
- [ ] TCP target group created
- [ ] Both EC2 instances registered
- [ ] Both targets show Healthy
- [ ] NLB DNS endpoint responds successfully
- [ ] Traffic distribution tested
- [ ] Cleanup completed
