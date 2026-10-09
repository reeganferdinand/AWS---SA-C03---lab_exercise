# SAA-C03 Lab 03: Application Load Balancer – Distribute Traffic Across EC2 Instances

## 🎯 Tasks
1. Launch two EC2 instances running a sample web application
2. Create a security group for the ALB
3. Create a target group and register both instances
4. Create an internet-facing Application Load Balancer
5. Verify traffic distribution and health checks
6. Stop one instance and test failover
7. Clean up resources

---

## 🧩 Step 1: Launch Two EC2 Instances

1. Go to **EC2 → Instances → Launch instances**
2. Configure the first instance:

- **Name:** `saa-alb-ec2-01`
- **AMI:** Amazon Linux
- **Instance type:** A suitable small instance type
- **VPC:** Default VPC
- **Subnet:** First Availability Zone

3. Launch the second instance as `saa-alb-ec2-02` in a different Availability Zone.
4. Configure an EC2 security group named `saa-alb-ec2-sg-01`.
5. For this basic demo, allow inbound HTTP on port 80 so you can verify the application before restricting access in Lab 04.

Under **Advanced details → User data**, enter:

```bash
#!/bin/bash
yum install -y httpd
systemctl enable --now httpd
echo "Hello from $(hostname -f)" > /var/www/html/index.html
```

6. Launch both instances and wait until they are running.
7. Open each instance's public IPv4 address in a browser and verify the response.

---

## 🔹 Step 2: Create the ALB Security Group

1. Go to **EC2 → Security Groups → Create security group**
2. Configure:

- **Name:** `saa-alb-sg-01`
- **VPC:** Same VPC as EC2

3. Add the inbound rule:

- **Type:** HTTP
- **Port:** 80
- **Source:** `0.0.0.0/0`

4. Create the security group.

👉 This allows HTTP requests to reach the internet-facing ALB.

---

## 🔹 Step 3: Create a Target Group

1. Go to **EC2 → Target Groups → Create target group**
2. Configure:

- **Target type:** Instances
- **Name:** `saa-alb-tg-01`
- **Protocol:** HTTP
- **Port:** 80
- **VPC:** Same VPC as EC2
- **Health check path:** `/`

3. Register both EC2 instances on port 80.
4. Create the target group.

Wait until both targets become **Healthy**.

---

## 🔹 Step 4: Create an Application Load Balancer

1. Go to **EC2 → Load Balancers → Create load balancer**
2. Select **Application Load Balancer**.
3. Configure:

- **Name:** `saa-alb-01`
- **Scheme:** Internet-facing
- **IP address type:** IPv4
- **VPC:** Same VPC as EC2
- **Mappings:** Select subnets in at least two Availability Zones
- **Security group:** `saa-alb-sg-01`

4. Under **Listeners and routing**, configure:

- **Protocol:** HTTP
- **Port:** 80
- **Default action:** Forward to `saa-alb-tg-01`

5. Create the load balancer.
6. Wait until its state is **Active**.

---

## 🔹 Step 5: Test Load Balancing

1. Open `saa-alb-01` and copy its DNS name.
2. Open the DNS name in your browser:

```text
http://<ALB-DNS-NAME>/
```

3. Refresh the page several times.
4. Verify that responses can come from both EC2 instances.
5. Go to **Target Groups → `saa-alb-tg-01` → Targets**.
6. Confirm both targets are **Healthy**.

---

## 🔹 Step 6: Test Health Checks and Recovery

1. Go to **EC2 → Instances**.
2. Stop `saa-alb-ec2-01`.
3. Wait for its target health status to change.
4. Refresh the ALB DNS endpoint and verify that traffic is served by the remaining healthy target.
5. Start `saa-alb-ec2-01` again.
6. Wait until its target status returns to **Healthy**.
7. Verify that both instances are eligible to receive traffic again.

👉 The ALB uses target health checks to avoid routing requests to unhealthy targets.

---

## 🧹 Step 7: Cleanup

1. Delete `saa-alb-01`.
2. Delete `saa-alb-tg-01`.
3. Terminate `saa-alb-ec2-01`.
4. Terminate `saa-alb-ec2-02`.
5. Delete `saa-alb-sg-01` and `saa-alb-ec2-sg-01` if they are no longer in use.

⚠️ ALBs incur charges while provisioned. Clean up promptly after the lab.

---

## ✅ Verification

- [ ] Two EC2 web servers launched
- [ ] Target group created
- [ ] Both targets show Healthy
- [ ] ALB DNS endpoint works
- [ ] Traffic reaches healthy targets
- [ ] Recovery tested
- [ ] Cleanup completed
