# AWS SAA-C03 Hands-on Lab: Encrypt an Existing EBS Volume

**Domain:** Design Secure Architectures / Design High-Performing Architectures  
**Topic:** Amazon EBS encryption, snapshots, and KMS  
**Status:** Not Started  
**Estimated time:** 30–45 minutes  
**Cost level:** Low, but EBS volumes and snapshots can incur charges while they exist.

---

## 1. Learning objectives

By the end of this lab, you should be able to:

- Create and identify an unencrypted EBS volume.
- Create a snapshot from an unencrypted volume.
- Copy the snapshot and enable encryption with an AWS KMS key.
- Create an encrypted volume from the encrypted snapshot.
- Explain the shortcut: create an encrypted volume directly from an unencrypted snapshot, when the console/API supports that option.
- Verify encryption status and clean up the resources.

## 2. Key concepts for the exam

When an EBS volume is encrypted, AWS provides encryption at rest, encryption for data moving between the EC2 instance and the volume, and encryption for snapshots created from that encrypted volume. Volumes restored from encrypted snapshots are encrypted too. Encryption and decryption are handled transparently by AWS.

- EBS encryption uses AWS KMS keys and industry-standard encryption (AES-256).
- Encrypting an existing unencrypted volume generally involves creating a snapshot and creating a new encrypted volume from it.
- Copying an unencrypted snapshot and enabling encryption creates an encrypted snapshot.
- An encrypted snapshot produces encrypted volumes.
- You cannot turn encryption on in-place for an existing unencrypted EBS volume.
- Default EBS encryption settings can vary by account and Region. Confirm the actual encryption status instead of assuming a new volume is unencrypted.

## 3. Prerequisites and safety

- Sign in to the AWS Management Console.
- Choose one Region and keep all lab resources in that Region.
- Use a lab account or an account where you are authorized to create EBS resources.
- Required permissions typically include EC2 volume/snapshot create, describe, and delete actions, plus KMS key usage permissions as applicable.
- Use a small **1 GiB** General Purpose SSD volume (for example, `gp3`) to keep the lab small.
- Do **not** detach or modify a production volume. This lab uses a new disposable volume.
- If your account enforces EBS encryption by default, you may not be able to create an unencrypted volume. In that case, use the optional exam discussion below and do not weaken account-wide security settings just to reproduce the demo.

### Resource naming

Use these names to make cleanup easier:

- Original volume: `saa-ebs-unencrypted-lab`
- Original snapshot: `saa-ebs-unencrypted-snapshot`
- Encrypted copied snapshot: `saa-ebs-encrypted-snapshot`
- Replacement volume: `saa-ebs-encrypted-volume`

Record the IDs as you go:

| Resource | ID | Status |
|---|---|---|
| Original volume |  | Not created |
| Original snapshot |  | Not created |
| Encrypted snapshot copy |  | Not created |
| Encrypted replacement volume |  | Not created |

---

## 4. Lab A — Create an unencrypted EBS volume

1. Open **EC2 → Elastic Block Store → Volumes**.
2. Select **Create volume**.
3. Choose:
   - Availability Zone: choose one in your selected Region.
   - Volume type: `gp3`.
   - Size: `1 GiB`.
   - Encryption: leave disabled **only if your account permits this and policy allows it**.
4. Add the name tag `saa-ebs-unencrypted-lab`.
5. Create the volume and wait until its state is **Available**.
6. Inspect the **Encryption** column/details and record whether it is encrypted.

**Verification checkpoint**

- [ ] Volume exists and is `Available`.
- [ ] Volume ID recorded.
- [ ] Encryption status recorded.

**Important:** If encryption is forced on by default, record that result and continue with an existing authorized unencrypted lab volume only if one is available. Do not disable default encryption account-wide.

## 5. Lab B — Create a snapshot

1. In **EC2 → Volumes**, select the original lab volume.
2. Choose **Actions → Create snapshot**.
3. Add a description such as `SAA-C03 unencrypted EBS lab snapshot`.
4. Create the snapshot.
5. Open **EC2 → Elastic Block Store → Snapshots**.
6. Wait until the snapshot status is **Completed**.
7. Inspect and record the snapshot's **Encryption** status.

**Verification checkpoint**

- [ ] Snapshot completed.
- [ ] Snapshot ID recorded.
- [ ] Encryption status checked.

Expected result for an unencrypted source volume: the snapshot is unencrypted.

## 6. Lab C — Copy the snapshot and enable encryption

1. In **Snapshots**, select the original snapshot.
2. Choose **Actions → Copy snapshot**.
3. Keep the destination Region the same as the source Region for this lab.
4. Enable **Encrypt this snapshot**.
5. Choose the default AWS managed EBS KMS key (commonly shown as `aws/ebs`) if available and appropriate for your account. A customer managed key may be used if you have permission and a reason to manage the key yourself.
6. Add a description such as `SAA-C03 encrypted EBS lab snapshot`.
7. Copy the snapshot.
8. Wait for the copied snapshot to reach **Completed**.
9. Check its **Encryption** status and the KMS key shown.

**Verification checkpoint**

- [ ] Copied snapshot completed.
- [ ] Copied snapshot ID recorded.
- [ ] Encryption is **Enabled**.
- [ ] KMS key recorded.

## 7. Lab D — Create an encrypted volume from the encrypted snapshot

1. Select the encrypted snapshot copy.
2. Choose **Actions → Create volume from snapshot**.
3. Select the same Availability Zone as the EC2 instance if you intend to attach it to that instance later. For this standalone lab, choose an Availability Zone and note it.
4. Use `gp3` and a size at least as large as the source volume (1 GiB for this lab).
5. Confirm encryption is enabled and the intended KMS key is selected.
6. Tag the volume `saa-ebs-encrypted-volume`.
7. Create the volume and wait until it is **Available**.
8. Inspect the volume details.

**Verification checkpoint**

- [ ] New volume is `Available`.
- [ ] New volume's encryption status is **Encrypted**.
- [ ] KMS key recorded.
- [ ] New volume ID recorded.

## 8. Lab E — Explore the shortcut

The lecture also demonstrates creating an encrypted volume directly from an unencrypted snapshot, where the console/API offers the encryption option during volume creation.

1. Select the original unencrypted snapshot.
2. Choose **Actions → Create volume from snapshot**.
3. Check whether the creation form lets you enable encryption and select a KMS key.
4. **Do not create a second volume unless you want the extra practice and remember to delete it.**
5. Record whether the option is available in your Region/account and explain how the resulting volume's encryption differs from the source snapshot's status.

**Expected learning:** An unencrypted snapshot can be used as the source for a newly created encrypted volume when the supported creation flow permits it. The source snapshot itself remains unencrypted; the new volume is encrypted.

## 9. Optional CLI verification

Use the AWS CLI only if it is configured for the same account and Region. Replace the placeholders with your actual IDs.

```bash
aws ec2 describe-volumes \
  --volume-ids vol-REPLACE_ME \
  --query 'Volumes[0].{VolumeId:VolumeId,State:State,Encrypted:Encrypted,KmsKeyId:KmsKeyId,AvailabilityZone:AvailabilityZone}' \
  --output table
```

```bash
aws ec2 describe-snapshots \
  --snapshot-ids snap-REPLACE_ME \
  --query 'Snapshots[0].{SnapshotId:SnapshotId,State:State,Encrypted:Encrypted,KmsKeyId:KmsKeyId}' \
  --output table
```

For an encrypted snapshot copy, the expected value for `Encrypted` is `True`. For the encrypted replacement volume, `Encrypted` should also be `True`.

## 10. Troubleshooting notes

- **Cannot create an unencrypted volume:** EBS encryption by default may be enabled, or an organizational policy may require encryption. Do not weaken security controls just to match a tutorial.
- **KMS access denied:** Check IAM permissions and the KMS key policy. A customer managed key may require explicit access.
- **Snapshot is pending:** Wait for it to reach `Completed` before creating dependent resources.
- **Cannot attach a volume to an instance:** EBS volumes must be in the same Availability Zone as the instance.
- **Volume is still creating:** Wait until its state becomes `Available` before attaching it.
- **Unexpected encryption status:** Inspect the volume/snapshot details and KMS key, and confirm the Region/account you are viewing.

## 11. SAA-C03 exam checks

Try answering these without looking at the notes:

1. Can an existing unencrypted EBS volume be encrypted in place?
2. What workflow can be used to migrate an unencrypted volume to an encrypted one?
3. What happens to snapshots created from an encrypted volume?
4. What happens to a volume created from an encrypted snapshot?
5. Does copying an unencrypted snapshot with encryption enabled encrypt the original snapshot?
6. Who handles encryption and decryption during normal EBS use?
7. What should you check if encryption is forced on in your account?
8. Why must an EBS volume be in the same Availability Zone as an EC2 instance before attachment?

### Answer key

1. No; create an encrypted replacement using a snapshot-based workflow.
2. Create a snapshot, copy it with encryption enabled (or use a supported flow to create an encrypted volume from the snapshot), then create a new encrypted volume and migrate/attach as appropriate.
3. Snapshots of an encrypted volume are encrypted.
4. The restored volume is encrypted.
5. No; the copy is encrypted, while the original snapshot remains unchanged.
6. AWS handles it transparently using KMS-backed encryption.
7. Respect the account/organization security configuration; don't disable it solely to reproduce a tutorial.
8. EBS volumes are Availability Zone scoped.

## 12. Cleanup — avoid ongoing charges

Delete only resources created for this lab. Double-check names and IDs before deleting.

1. If you created or attached a volume to an instance, detach it first when appropriate. Never detach a production/root volume for this lab.
2. Delete the encrypted replacement volume after it is no longer needed.
3. Delete any optional extra volume created in Lab E.
4. Delete the encrypted copied snapshot.
5. Delete the original snapshot.
6. Delete the original volume.
7. Revisit **Volumes** and **Snapshots** and confirm the lab resources are gone.

**Cleanup checklist**

- [ ] Encrypted replacement volume deleted.
- [ ] Optional extra volume deleted (if created).
- [ ] Encrypted snapshot copy deleted.
- [ ] Original snapshot deleted.
- [ ] Original volume deleted.
- [ ] No lab resources remain.

Deleting a snapshot or volume is destructive. Verify each resource ID before deletion. KMS keys managed by AWS for EBS are not something you need to delete for this lab.

## 13. Completion record

| Task | Status | Notes / screenshot filename |
|---|---|---|
| Create/inspect source volume | Not Started | |
| Create source snapshot | Not Started | |
| Copy snapshot with encryption enabled | Not Started | |
| Create encrypted replacement volume | Not Started | |
| Explore direct encrypted-volume shortcut | Not Started | |
| Verify using console or CLI | Not Started | |
| Answer exam questions | Not Started | |
| Clean up resources | Not Started | |

**Evidence to capture:** source volume encryption status, original snapshot status, encrypted copied snapshot details, encrypted replacement volume details, and final cleanup confirmation.

---

**Lab complete when:** You can explain the snapshot-copy workflow, prove the replacement volume is encrypted, answer the exam checks, and confirm all temporary resources are cleaned up.
