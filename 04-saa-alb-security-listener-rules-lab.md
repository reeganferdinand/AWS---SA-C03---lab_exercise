# SAA-C03 Lab 04: Application Load Balancer – Security Groups and Listener Rules

## 🎯 Tasks
1. Restrict EC2 HTTP access to the ALB security group only
2. Verify direct EC2 access is blocked
3. Create an ALB listener rule for `/error`
4. Return a custom HTTP 404 response
5. Test the rule and clean up

---

## 🧩 Step 1: Update the EC2 Security Group

1. Go to **EC2 → Security Groups**.
2. Select `saa-alb-ec2-sg-01`.
3. Choose **Inbound rules → Edit inbound rules**.
4. Remove the HTTP rule allowing traffic from `0.0.0.0/0`.
5. Add a new rule:

- **Type:** HTTP
- **Port:** 80
- **Source:** Security group → `saa-alb-sg-01`

6. Save the rules.

👉 Now HTTP traffic to the EC2 instances is allowed only from resources using the ALB security group.

---

## 🔹 Step 2: Verify Network Security

1. Try accessing an EC2 public IPv4 address directly.
2. Verify direct HTTP access fails.
3. Open the ALB DNS endpoint:

```text
http://<ALB-DNS-NAME>/
```

4. Confirm the web application still works.
5. Go to **Target Groups → `saa-alb-tg-01` → Targets**.

👉 Both targets should remain **Healthy**.

---

## 🔹 Step 3: Create an ALB Listener Rule

1. Go to **EC2 → Load Balancers**.
2. Select `saa-alb-01`.
3. Open **Listeners and rules**.
4. Select the HTTP listener on port 80.
5. Click **Add rule**.

Configure:

- **Rule name:** `saa-alb-rule-01`
- **Condition:** Path
- **Path pattern:** `/error`
- **Priority:** `5`

---

## 🔹 Step 4: Configure a Fixed Response

Under the rule's actions, select **Return fixed response**.

Configure:

- **Response code:** `404`
- **Response body:** `Not found, custom error`
- **Content type:** `text/plain`

Save the rule.

👉 The ALB returns the fixed response when the request matches `/error`, without forwarding that request to an EC2 target.

---

## 🔹 Step 5: Test the Listener Rule

Open the ALB DNS name in your browser.

**Normal request:**

```text
http://<ALB-DNS-NAME>/
```

Expected: Response from an EC2 web server.

**Custom error request:**

```text
http://<ALB-DNS-NAME>/error
```

Expected: HTTP `404` with the message:

```text
Not found, custom error
```

Verify that the normal application request still works.

---

## 🧹 Step 6: Cleanup

1. Remove `saa-alb-rule-01` if you want to restore the original listener behavior.
2. If you are finishing the ALB practical, delete `saa-alb-01`.
3. Delete `saa-alb-tg-01`.
4. Terminate `saa-alb-ec2-01` and `saa-alb-ec2-02`.
5. Delete the lab security groups after confirming they are no longer in use.

⚠️ This lab extends Lab 03. Reuse its resources rather than creating duplicates.

---

## ✅ Verification

- [ ] EC2 HTTP access restricted to the ALB security group
- [ ] Direct EC2 access blocked
- [ ] ALB endpoint still works
- [ ] `/error` returns HTTP 404
- [ ] Target health verified
- [ ] Cleanup completed
