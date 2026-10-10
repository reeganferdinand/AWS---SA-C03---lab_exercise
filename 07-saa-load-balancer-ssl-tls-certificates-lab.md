# SAA-C03 Lab 07: AWS Load Balancers – Configure SSL/TLS Certificates


## 🎯 Aim

To understand how to configure HTTPS/TLS listeners on Application Load Balancers (ALB) and Network Load Balancers (NLB) using SSL/TLS certificates.

## 🧰 AWS Services Used

- Amazon EC2
- Application Load Balancer (ALB)
- Network Load Balancer (NLB)
- AWS Certificate Manager (ACM)
- IAM (alternative certificate source)

## 🏗️ Architecture Diagram

```text
               Clients / Browser
                       |
                    HTTPS
                    Port 443
                       |
              +------------------+
              |       ALB        |
              |  TLS Certificate |
              +------------------+
                       |
                 HTTP : 80
                       |
                Target Group
                  /       \
                 v         v
             EC2-01      EC2-02
```

*Note: The certificate is configured on the load balancer listener. The diagram illustrates the ALB setup; an NLB can use a TLS listener similarly.*

## 🧪 Procedure

### Part A: Configure HTTPS on an ALB

1. Open the **EC2 Console → Load Balancers**.
2. Select the existing ALB `saa-alb-01`.
3. Open the **Listeners** tab and choose **Add listener**.
4. Configure the listener:
   - **Protocol:** HTTPS
   - **Port:** 443
   - **Default action:** Forward to `saa-alb-tg-01`
5. Configure the SSL/TLS security policy. Keep the default policy for this lab.
6. Select a certificate source:
   - **ACM:** Choose an available certificate.
   - **IAM:** Select a certificate stored in IAM, if available.
   - **Import:** Import a certificate into ACM using the certificate body, private key, and certificate chain, as applicable.
7. Save the listener configuration.

**Important:** A valid certificate for the domain is required for normal browser trust. For a public website, ACM is the recommended certificate-management option. You generally need a domain name and DNS validation to obtain a publicly trusted certificate.

### Part B: Configure TLS on an NLB

1. Open **EC2 Console → Load Balancers**.
2. Select the existing NLB `saa-nlb-01`.
3. Open **Listeners** and choose **Add listener**.
4. Configure the listener:
   - **Protocol:** TLS
   - **Port:** 443
   - **Default action:** Forward to the appropriate target group, such as `saa-nlb-tg-01`.
5. Select the required TLS security policy.
6. Choose the certificate source: ACM, IAM, or import a certificate into ACM.
7. Review **Application-Layer Protocol Negotiation (ALPN)** settings if needed. Leave the default unless the application requires a specific configuration.
8. Save the listener.

## ✅ Verification

- Confirm that the HTTPS listener on the ALB uses port 443.
- Confirm that the TLS listener on the NLB uses port 443.
- Verify that each listener forwards traffic to the intended target group.
- If a valid certificate and matching DNS record are configured, test the domain using `https://your-domain`.
- Check the load balancer's target health to ensure backend instances are healthy.

## 📝 Key Exam Notes

| Feature | ALB | NLB |
|---|---|---|
| Secure listener | HTTPS | TLS |
| Default secure port | 443 | 443 |
| Certificate selection | ACM, IAM, or import into ACM | ACM, IAM, or import into ACM |
| Security policy | Configures TLS negotiation | Configures TLS negotiation |
| Main role | Application-layer HTTP/HTTPS routing | Network-layer traffic handling with TLS support |

- **ACM** manages SSL/TLS certificates for AWS services.
- **HTTPS on ALB** and **TLS on NLB** are the listener protocols to remember.
- A TLS security policy determines supported TLS versions and cipher suites.
- ALPN negotiates application protocols over TLS, such as HTTP/2.
- A certificate on the load balancer secures the client-to-load-balancer connection. The connection from the load balancer to the targets is configured separately.

## 🧹 Cleanup

- Remove the HTTPS listener from the ALB if it is no longer needed.
- Remove the TLS listener from the NLB if it is no longer needed.
- Do not delete certificates shared by other resources.
- Keep the existing ALB, NLB, target groups, and EC2 instances if they will be reused in upcoming labs.

## 🏁 Result

Successfully reviewed the configuration of HTTPS listeners on an ALB and TLS listeners on an NLB, including certificate selection, TLS security policies, and ALPN settings.
