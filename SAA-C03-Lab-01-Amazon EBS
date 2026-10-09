# SAA-C03 Lab 01: Amazon EBS – Encrypt an Existing EBS Volume

## 🎯 Tasks
1. Create an **unencrypted EBS volume**
2. Take a **snapshot** of the volume
3. Copy the snapshot with **encryption enabled using AWS KMS**
4. Create a new **encrypted EBS volume**
5. Verify encryption and clean up resources

---

## 🧩 Step 1: Create an EBS Volume

1. Go to **AWS Console → EC2 → Elastic Block Store → Volumes**
2. Click **Create volume**
3. Configure:

- **Volume type:** `gp3`
- **Size:** `1 GiB`
- **Availability Zone:** Choose one in your Region
- **Encryption:** Disabled, only if account settings and security policies allow it

4. Add the name tag: `saa-ebs-volume-01`
5. Click **Create volume**
6. Wait until the state becomes **Available**

👉 If encryption is enforced by default, do not disable account-wide encryption just for this lab.

---

## 🔹 Step 2: Create a Snapshot

1. Select `saa-ebs-volume-01`
2. Click **Actions → Create snapshot**
3. Enter the description: `SAA-C03 EBS encryption lab`
4. Create the snapshot
5. Go to **Snapshots** and wait until the status is **Completed**

Name the snapshot:

`saa-ebs-snap-01`

Verify its encryption status.

---

## 🔹 Step 3: Copy the Snapshot with Encryption

1. Select `saa-ebs-snap-01`
2. Click **Actions → Copy snapshot**
3. Keep the destination Region the same
4. Enable **Encrypt this snapshot**
5. Select the appropriate EBS KMS key, such as `aws/ebs` if available
6. Copy the snapshot and wait until it is **Completed**

Name the encrypted copy:

`saa-ebs-snap-02`

👉 Verify that the copied snapshot shows **Encrypted: Yes**.

---

## 🔹 Step 4: Create an Encrypted Volume

1. Select `saa-ebs-snap-02`
2. Click **Actions → Create volume from snapshot**
3. Configure:

- **Volume name:** `saa-ebs-volume-02`
- **Volume type:** `gp3`
- **Size:** At least the original volume size
- **Availability Zone:** Choose the required AZ
- **Encryption:** Enabled

4. Create the volume
5. Wait until it becomes **Available**

👉 Open the volume details and confirm **Encrypted: Yes**.

---

## 🔹 Step 5: Explore the Shortcut

1. Select the original unencrypted snapshot.
2. Click **Actions → Create volume from snapshot**.
3. Check whether the form allows encryption to be enabled directly.
4. If available, create an encrypted volume and verify its status.

The original snapshot remains unencrypted; the newly created volume can be encrypted.

---

## 🧹 Step 6: Cleanup

1. Delete `saa-ebs-volume-02`.
2. Delete any optional volume created in Step 5.
3. Delete `saa-ebs-snap-02`.
4. Delete `saa-ebs-snap-01`.
5. Delete `saa-ebs-volume-01`.

⚠️ Delete only resources created for this lab.

---

## ✅ Verification

- [ ] Original volume encryption status checked
- [ ] Snapshot created successfully
- [ ] Encrypted snapshot copy created
- [ ] New volume shows **Encrypted: Yes**
- [ ] Cleanup completed
