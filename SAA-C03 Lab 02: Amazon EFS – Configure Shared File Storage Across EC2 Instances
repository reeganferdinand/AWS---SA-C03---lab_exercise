Lab Exercise: Amazon EFS Shared File System (SAA-C03)
Tasks
1. Create a Regional EFS file system with encryption enabled.
2. Configure mount targets and security groups for EFS access.
3. Launch two EC2 instances in different Availability Zones.
4. Mount the same EFS file system on both instances.
5. Create a file on Instance A and verify it is accessible from Instance B.
6. Clean up all AWS resources.

Step 1: Create an EFS File System
1. Go to AWS Console → EFS → Create file system → Customize.
2. Configure:
   - Name: saa-efs-demo
   - VPC: Default VPC
   - File system type: Regional
   - Automatic backups: Enabled
   - Lifecycle management: Move infrequently accessed files to IA after 30 days (optional)
   - Encryption at rest: Enabled
   - Throughput mode: Elastic
   - Performance mode: General Purpose
3. Configure network access using the selected VPC and its subnets.
4. Create the file system and wait until it becomes Available.

Step 2: Configure Security Groups
1. Go to EC2 → Security Groups → Create security group.
2. Create saa-efs-demo-sg in the same VPC.
3. Attach this security group to the EFS mount targets.
4. Configure the EFS security group inbound rule:
   - Type: NFS
   - Protocol/Port: TCP 2049
   - Source: Security group attached to your EC2 instances.
5. Ensure the EC2 security group allows outbound traffic to EFS on TCP 2049.

Step 3: Launch Two EC2 Instances
1. Go to EC2 → Instances → Launch instances.
2. Launch two Amazon Linux instances:
   - Names: saa-efs-instance-a and saa-efs-instance-b
   - Instance type: A small eligible instance type
   - VPC: Same VPC as EFS
   - Subnet: Place each instance in a different Availability Zone
   - Security group: Allow access to EFS on TCP 2049
3. Ensure both instances can be accessed through EC2 Instance Connect or SSH.

Step 4: Mount EFS on Both Instances
1. Open the EFS file system and select Attach.
2. Follow the recommended mount instructions for your instance's operating system.
3. On each Amazon Linux instance, install the EFS mount helper if needed:
sudo dnf install -y amazon-efs-utils


4. Create a mount point and mount the file system using its actual EFS ID:
sudo mkdir -p /mnt/efs
sudo mount -t efs -o tls fs-REPLACE_WITH_EFS_ID:/ /mnt/efs


Repeat on both instances. Ensure the EFS mount targets and security groups are correctly configured.

Step 5: Verify Shared Storage
On Instance A:
echo "Hello from Instance A" | sudo tee /mnt/efs/hello.txt
cat /mnt/efs/hello.txt


On Instance B:
cat /mnt/efs/hello.txt


✅ Instance B should display Hello from Instance A, confirming both instances share the same EFS file system.

Step 6: Cleanup
1. Unmount EFS on both instances:
sudo umount /mnt/efs


2. Terminate both EC2 instances.
3. Delete the EFS file system.
4. Delete any security groups created specifically for this lab, after confirming they are no longer in use.
Note: EFS storage, mount targets, and EC2 instances may incur charges. Verify that the resources have been deleted.
