#  Build a Private CA with OpenSSL — Secure Internal Services

## 📘 Summary

This project demonstrates how I built a **Private Certificate Authority (CA)** using OpenSSL on Kali Linux to solve a common security problem: **securing internal communications** using SSL/TLS when external certificate providers aren't feasible.

I created a CA, generated certificates for a hypothetical internal app, and verified the setup — all using cryptographic best practices.

---

##  Real-World Security Problem

### The Risk:
Many internal applications transmit sensitive data over **plain HTTP**, making them vulnerable to **Man-in-the-Middle attacks**. This is often because:

- Developers assume internal networks are “safe”.
- Organizations want to avoid paying for public CA certificates.

###  The Solution:
Implement **PKI** internally:
- Create a **private CA** to sign certificates for internal use.
- Use these certificates to enable **HTTPS** encryption for apps and services.
- Trust the CA in all client machines and browsers within the organization.

---

##  My Thought Process & Commands

### Step 1: Create a working directory
> I like keeping things organized, especially when dealing with sensitive files.
```bash
mkdir ~/Desktop/PKI && cd ~/Desktop/PKI

### Step 2: Generate CA private key (2048-bit RSA, AES-256 encrypted)
> The CA's private key must be secure and password-protected.
```bash
openssl genpkey -algorithm RSA -out ca_private_key.pem -aes256 -pass pass:altschool

### Step 3: Create the CA’s self-signed certificate
> I filled in details like country, organization, etc., to reflect the issuer.
```bash
openssl req -x509 -new -key ca_private_key.pem -days 365 -out ca_certificate.pem -passin pass:altschool

### Step 4: Generate user private key
> This represents the private key for the internal app/service.
```bash
openssl genpkey -algorithm RSA -out user_private_key.pem -aes256 -pass pass:altschool

### Step 5: Generate CSR (Certificate Signing Request)
> This is like asking the CA: “Can you vouch for me?” with my public key and ID.
```bash
openssl req -new -key user_private_key.pem -out user_csr.pem -passin pass:altschool

### Step 6:  Inspect the CSR (debug step)
> Always verify what's inside. Details should match expected fields.
```bash
openssl req -text -noout -verify -in user_csr.pem

### Step 7: Sign the CSR with your CA (optional but recommended)
> This issues a certificate that clients can trust if they trust your CA.
```bash
openssl x509 -req -in user_csr.pem -CA ca_certificate.pem -CAkey ca_private_key.pem \
-CAcreateserial -out user_certificate.pem -days 365 -sha256 -passin pass:altschool




 Outcome
- The CA’s certificate can be installed on all internal systems.

- The user_certificate.pem can now be used to enable HTTPS for internal services.

- This solution avoids third-party CA costs while eliminating MITM risks.

