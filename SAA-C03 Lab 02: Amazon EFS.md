# SAA-C03 Lab 02: Amazon EFS – Shared File Storage Across EC2 Instances

## 🎯 Tasks
1. Create a **Regional EFS file system**
2. Configure EFS security groups and mount targets
3. Launch two EC2 instances in different Availability Zones
4. Mount EFS on both instances
5. Create a file on one instance and read it from the other
6. Clean up resources

---

## 🧩 Step 1: Create an EFS File System

1. Go to **AWS Console → EFS → Create file system → Customize**
2. Configure:

- **Name:** `saa-efs-efs-01`
- **VPC:** Default VPC
- **File system type:** Regional
- **Automatic backups:** Enabled
- **Encryption at rest:** Enabled
- **Throughput mode:** Elastic
- **Performance mode:** General Purpose

3. Configure lifecycle management if desired.
4. Under network access, select the VPC and subnets in at least two Availability Zones.
5. Create the file system and wait until it is **Available**.

---

## 🔹 Step 2: Configure Security Groups

1. Go to **EC2 → Security Groups → Create security group**
2. Configure:

- **Name:** `saa-efs-sg-01`
- **VPC:** Same VPC as EFS and EC2

3. Attach this security group to the EFS mount targets.
4. Configure the EFS security group's inbound rule:

- **Type:** NFS
- **Protocol:** TCP
- **Port:** `2049`
- **Source:** Security group attached to the EC2 instances

5. Ensure the EC2 security group permits outbound traffic to EFS on TCP 2049.

---

## 🔹 Step 3: Launch Two EC2 Instances

1. Go to **EC2 → Instances → Launch instances**
2. Configure the first instance:

- **Name:** `saa-efs-ec2-01`
- **AMI:** Amazon Linux
- **Instance type:** A suitable small instance type
- **VPC:** Same VPC as EFS
- **Subnet:** First Availability Zone
- **Security group:** Allow access to EFS on TCP 2049

3. Launch the second instance:

- **Name:** `saa-efs-ec2-02`
- **AMI:** Amazon Linux
- **Subnet:** A different Availability Zone
- **Security group:** Allow access to EFS on TCP 2049

4. Wait until both instances are running.

👉 Use EC2 Instance Connect or SSH to access each instance.

---

## 🔹 Step 4: Install the EFS Mount Helper

Run on **both EC2 instances**:

```bash
sudo dnf install -y amazon-efs-utils
```

Create the mount directory:

```bash
sudo mkdir -p /mnt/efs
```

Go to **EFS → File systems → `saa-efs-efs-01` → Attach**.

Copy the mount command and replace the placeholder below with your actual EFS ID:

```bash
sudo mount -t efs -o tls fs-REPLACE_WITH_EFS_ID:/ /mnt/efs
```

Run the command on both instances.

---

## 🔹 Step 5: Verify Shared Storage

### On Instance A

```bash
echo "Hello from Instance A" | sudo tee /mnt/efs/hello.txt
cat /mnt/efs/hello.txt
```

Expected output:

```text
Hello from Instance A
```

### On Instance B

```bash
cat /mnt/efs/hello.txt
```

Expected output:

```text
Hello from Instance A
```

🎉 The same file is accessible from both EC2 instances because they share the same EFS file system.

---

## 🧹 Step 6: Cleanup

1. Unmount EFS on both instances:

```bash
sudo umount /mnt/efs
```

2. Terminate `saa-efs-ec2-01`.
3. Terminate `saa-efs-ec2-02`.
4. Delete `saa-efs-efs-01`.
5. Delete `saa-efs-sg-01` after confirming it is no longer in use.

⚠️ EFS storage and mount targets can incur charges. Verify that the file system has been deleted.

---

## ✅ Verification

- [ ] Regional EFS created
- [ ] NFS port 2049 configured
- [ ] Two instances launched in different AZs
- [ ] EFS mounted on both instances
- [ ] File created on Instance A and read on Instance B
- [ ] Cleanup completed
