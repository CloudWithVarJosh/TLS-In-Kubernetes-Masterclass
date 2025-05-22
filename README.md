# TLS in Kubernetes MASTERCLASS | mTLS, CSR, CA, SSH, kubeconfig & Cluster Security Explained

## Video reference for the Masterclass is the following:

[![Watch the video](https://img.youtube.com/vi/J2Rx2fEJftQ/maxresdefault.jpg)](https://www.youtube.com/watch?v=J2Rx2fEJftQ&ab_channel=CloudWithVarJosh)

---
## ⭐ Support the Project  
If this **repository** helps you, give it a ⭐ to show your support and help others discover it! 

---

## Table of Contents

- [Introduction](#introduction)
- [Table of Contents](#table-of-contents)
- [What is Encryption? Why Do We Need It?](#what-is-encryption-why-do-we-need-it)
- [Two Main Types of Encryption](#two-main-types-of-encryption)
  - [1. Symmetric Encryption (Shared Key Encryption)](#1-symmetric-encryption-shared-key-encryption)
  - [2. Asymmetric Encryption (Public Key Cryptography)](#2-asymmetric-encryption-public-key-cryptography)
- [Encryption Approaches: In-Transit vs At-Rest](#encryption-approaches-in-transit-vs-at-rest)
  - [1. Encryption In-Transit](#1-encryption-in-transit)
  - [2. Encryption At-Rest](#2-encryption-at-rest)
- [Understanding Symmetric and Asymmetric Encryption Through Real-Life Scenarios](#understanding-symmetric-and-asymmetric-encryption-through-real-life-scenarios)
  - [Scenario 1: Symmetric Encryption (Disk Encryption)](#scenario-1-symmetric-encryption-disk-encryption)
  - [Scenario 2: Symmetric Encryption Alone Isn't Enough for Internet Communication](#scenario-2-symmetric-encryption-alone-isnt-enough-for-internet-communication)
  - [Scenario 3: Asymmetric Encryption (SSH into EC2 Instance)](#scenario-3-asymmetric-encryption-ssh-into-ec2-instance)
  - [Scenario 4: Combining Symmetric + Asymmetric (TLS Handshake)](#scenario-4-combining-symmetric--asymmetric-tls-handshake)
- [Public Key Cryptography](#public-key-cryptography)
  - [SSH Authentication (`ssh-keygen`)](#1-secure-remote-access-ssh-keygen-ssh-authentication)
  - [TLS Certificates and `openssl`](#2-secure-web-communication-openssl-tls-certificates--identity-validation)
- [What Public Key Cryptography (PKC) Provides](#what-public-key-cryptography-pkc-provides)
  - [Authentication (Identity Verification)](#1-authentication-identity-verification)
  - [Secure Key Exchange](#2-secure-key-exchange-foundation-for-encryption)
- [Types of TLS Certificate Authorities (CA)](#types-of-tls-certificate-authorities-ca-public-private-and-self-signed)
  - [Public CA](#public-ca)
  - [Private CA](#private-ca)
  - [Self-Signed Certificate](#self-signed-certificate)
- [Kubeconfig and Kubernetes Context](#kubeconfig-and-kubernetes-context)
  - [What is a Kubeconfig File and Why Do We Need It?](#what-is-a-kubeconfig-file-and-why-do-we-need-it)
  - [Kubernetes Context](#kubernetes-context)
  - [Example `kubeconfig` File](#example-kubeconfig-file)
  - [Common `kubectl config` Commands](#common-kubectl-config-commands-compact-view)
- [Mutual TLS (mTLS)](#mutual-tls-mtls)
  - [Private CAs in Kubernetes Clusters](#private-cas-in-kubernetes-clusters)
  - [Kubernetes Components as Clients and Servers](#kubernetes-components-as-clients-and-servers)
- [Understanding TLS in Kubernetes: No More Memorizing File Paths](#understanding-tls-in-kubernetes-no-more-memorizing-file-paths)
  - [Example 1: Scheduler (Client) ↔ API Server (Server)](#example-1-mtls-between-scheduler-client-and-api-server-server)
  - [Example 2: API Server (Client) ↔ etcd (Server)](#example-2-mtls-between-api-server-client-and-etcd-server)
  - [Example 3: API Server (Client) ↔ Kubelet (Server)](#example-3-mtls-between-api-server-client-and-kubelet-server)
- [Granting External Access to a New User Using Certificates and RBAC](#granting-cluster-access-to-a-new-user-seema-using-certificates-and-rbac)
- [Conclusion](#conclusion)
- [References](#references)

---

# TLS in Kubernetes MASTERCLASS — Full Series

## Introduction

This document is a consolidated walkthrough of the TLS in Kubernetes MASTERCLASS series. It merges Parts 1 through 5 into a single reference. Topics covered include the fundamentals of encryption, how Kubernetes uses TLS for secure component communication, understanding kubeconfig and contexts, and enabling external user access with CSR and RBAC.

---

Before we start discussing Kubernetes security topics like RBAC, Secrets, and TLS within clusters, it’s important to understand the foundation of secure communication: **encryption**.

---

## What is Encryption? Why Do We Need It?

**Encryption** is the process of converting readable data (plaintext) into unreadable data (ciphertext), ensuring that only authorized parties can read it.

We need encryption to:
- Protect sensitive data in transit (e.g., passwords, API tokens)
- Prevent eavesdropping on insecure networks
- Ensure confidentiality, integrity, and authenticity

---

## Two Main Types of Encryption

![Alt text](/images/30a.png)


### 1. Symmetric Encryption (Shared Key Encryption)

- Uses a **single secret key** for both **encryption** and **decryption**.
- This key must be **securely shared** between the sender and receiver.
- It is **fast, efficient**, and ideal for encrypting **large volumes of data**.
- The downside is: if the key is compromised, both encryption and decryption are at risk.

> In symmetric encryption, the key is commonly referred to as the **symmetric key**, **shared key**, or **secret key** — all indicating the same key is used for both encryption and decryption.


**Examples of Symmetric Encryption Algorithms:**
- **AES (Advanced Encryption Standard)** — the industry standard
- **ChaCha20** — optimized for low-power devices and mobile environments
- **Blowfish**, **Twofish**, and **DES** (older or legacy)

**Common Use Cases:**

- **Secure data transfer (post TLS handshake):**
  - Once the TLS handshake is complete and a session key is securely shared using asymmetric encryption, all communication (e.g., HTTPS) is encrypted using **symmetric encryption**.

- **Disk & storage encryption:**
  - **AWS EBS Volumes**, **Azure Managed Disks**, and **Google Persistent Disks** all use **symmetric encryption** under the hood (e.g., AES-256).
  - These cloud services often integrate with **Key Management Services (KMS)** to manage the encryption keys securely.

- **Encrypted backups and snapshots:**
  - Storing database or file system backups securely by encrypting the files before storing them on disk or in cloud object storage.

- **Database encryption:**
  - Symmetric encryption is commonly used for encrypting data at rest in relational and NoSQL databases (e.g., MySQL TDE, MongoDB encrypted storage engine).

- **Kubernetes Secrets encryption at rest**:
  - When encryption is configured in the kube-apiserver (e.g., using an AES-CBC or AES-GCM provider), Kubernetes encrypts Secrets, ConfigMaps, etc. using symmetric keys stored in a KMS or local file.


**Why Symmetric Encryption Is Preferred for Bulk Data:**

- It's **much faster** than asymmetric encryption, especially for large data sets or high-throughput systems.
- Easier to implement in hardware (e.g., hardware security modules or TPMs).
- When paired with strong key management (like KMS), it provides both performance and security.

---


### 2. Asymmetric Encryption (Public Key Cryptography)

![Alt text](/images/30a.png)

- Uses a **key pair**: a **public key** (shared with everyone) and a **private key** (kept secret).
- Data encrypted with the **public key** can only be decrypted with the **private key**, and vice versa.
- It's commonly used for **secure key exchange**, **authentication**, and **digital signatures**.
- While more **computationally expensive** than symmetric encryption, it's ideal for establishing trust and securely bootstrapping secure channels.

**Examples of Asymmetric Encryption Algorithms:**
- **Ed25519** — modern default for SSH keys and JWT signing in cloud-native systems
- **RSA** — still widely supported; commonly used in TLS certs, Kubernetes components, and legacy SSH
- **Elliptic Curve Cryptography (ECC)** — used in ECDSA; faster and more compact than RSA


**Common Use Cases:**

- **TLS Handshake (HTTPS):**
  - During the TLS handshake (e.g., accessing `https://github.com`), the server presents a **certificate containing its public key**.
  - The client uses this to securely exchange a **session key**, which is then used for symmetric encryption.

- **SSH Authentication:**
  - SSH clients authenticate using **asymmetric key pairs** (public and private SSH keys).
  - The server verifies your identity using the public key and grants access without a password.

- **Kubernetes x.509 Certificates:**
  - Components like `kube-apiserver`, `kubelet`, `etcd`, and `kubectl` use **client and server certificates** for mutual authentication.
  - These certificates are based on **RSA or ECC** public-private key cryptography.

- **Digital Signatures:**
  - Signing Git commits with **GPG/PGP**
  - **JWT tokens** used in OIDC and Kubernetes service account tokens are often signed using private keys and verified using public keys.

- **Secure Secrets Delivery:**
  - Tools like **Vault by HashiCorp** and **Sealed Secrets** use asymmetric encryption to encrypt sensitive data (e.g., a Secret YAML file).
  - Only the controller inside the cluster can decrypt it using the corresponding private key.

**Why Use Asymmetric Encryption?**

- You can **safely distribute** public keys — there's no risk if someone intercepts them.
- Ideal for use cases where two systems **don’t already share a secret**.
- Enables strong **identity verification** and **trust models**, such as PKI (Public Key Infrastructure).

**Key Considerations:**

- Slower than symmetric encryption — that’s why it’s mostly used during **initial connection setup** or for signing.
- Most modern systems use asymmetric encryption to **establish trust and exchange a symmetric key**, and then **switch to symmetric encryption** for speed.

---

## Summary Table

| Feature              | Symmetric Encryption        | Asymmetric Encryption             |
|----------------------|-----------------------------|-----------------------------------|
| Keys Used            | One shared secret key        | Public & private key pair         |
| Speed                | Fast                         | Slower                            |
| Security Risk        | Key sharing is risky         | Secure key exchange possible      |
| Use Case             | Bulk data encryption         | Identity/authentication, key exchange |
| Examples             | AES                          | RSA, TLS handshakes, SSH          |

---

### Symmetric and Asymmetric Encryption: Roles and Functions

**Symmetric encryption** algorithms, such as **AES (Advanced Encryption Standard)**, are highly efficient and fast, making them ideal for encrypting and decrypting large volumes of data. Due to their speed, symmetric encryption is typically used for bulk data encryption, such as protecting files, database records, or disk contents. These algorithms rely on a single shared secret key for both encryption and decryption, which ensures a streamlined and quick process, particularly when dealing with large datasets.

On the other hand, **asymmetric encryption algorithms** like **RSA (Rivest-Shamir-Adleman)** or **ECC (Elliptic Curve Cryptography)** play a crucial role in securing the exchange of keys used in symmetric encryption. While they are slower and computationally more expensive than symmetric encryption, they are essential for securely transmitting the symmetric keys over potentially insecure channels. Asymmetric encryption uses a pair of keys — a **public key** to encrypt the data and a **private key** to decrypt it — ensuring that only the intended recipient with the correct private key can access the symmetric key. This layered approach combines the speed of symmetric encryption with the security of asymmetric encryption, providing both efficiency and robust protection for sensitive data.

---

## Encryption Approaches: In-Transit vs At-Rest

When we talk about encryption in cloud and Kubernetes environments, it typically falls into **two categories**:

![Alt text](/images/30b.png)

### 1. **Encryption In-Transit**

This protects data **while it's moving** — between:
- Client and server (e.g., your browser and GitHub.com)
- Pod to Pod communication
- Kubernetes components (like kubelet <-> kube-apiserver)

**How it's implemented:**
- **TLS** (Transport Layer Security) is the most common protocol used.
- Kubernetes uses TLS extensively — for `kubectl`, Ingress, the API server, etc.
- In service meshes (like Istio), **mTLS (mutual TLS)** is used to encrypt **Pod-to-Pod** traffic automatically.

**Why it matters:**
- Prevents eavesdropping, man-in-the-middle attacks, and session hijacking.
- Critical in multi-tenant clusters or when apps talk over public networks.

---

### 2. **Encryption At-Rest**

This protects data **when it’s stored** — in:
- Persistent volumes (EBS, EFS, PVs)
- Databases
- etcd (Kubernetes' backing store)
- Secrets stored on disk

**How it's implemented:**
- Data is encrypted using **symmetric encryption algorithms** like AES-256.
- Kubernetes supports encryption at rest for resources like **Secrets**, **ConfigMaps**, and **PersistentVolumeClaims** via encryption providers configured on the API server.

**Why it matters:**
- If someone gains access to your storage (EBS snapshot, physical disk), they can't read the raw data.
- Especially important for **compliance** (e.g., GDPR, HIPAA, PCI-DSS).

---

### Quick Comparison:

| Aspect             | Encryption In-Transit         | Encryption At-Rest              |
|--------------------|-------------------------------|---------------------------------|
| Purpose            | Secures data during transfer  | Secures data when stored        |
| Technology Used    | TLS, mTLS, VPN                | AES, KMS, disk encryption       |
| Common Tools       | TLS certs, Istio, NGINX       | KMS (AWS/GCP), Kubernetes providers |
| Example            | kubectl to API server         | etcd encryption, S3 object encryption |
| Who handles it?    | Network-level, sidecars       | Storage layer, cloud provider   |



> A secure system uses **both** encryption in-transit **and** at-rest.  
> Kubernetes and cloud-native tools provide support for both — but as DevOps/cloud engineers, it's your job to configure and validate them properly.

---

## Understanding Symmetric and Asymmetric Encryption Through Real-Life Scenarios

To make encryption concepts clear, let's walk through four practical scenarios involving a user named **Seema**. Each example builds upon the previous to gradually explain how encryption works — starting from her personal laptop to secure cloud communication.

---

### **Scenario 1: Symmetric Encryption (Disk Encryption)**  
**Use Case:** Protecting Seema's Laptop Hard Disk  
**Technology Example:** BitLocker (Windows)

![Alt text](/images/30c.png)

Seema has a Windows laptop with **BitLocker** enabled to encrypt her hard disk. BitLocker uses **symmetric encryption**, typically **AES (Advanced Encryption Standard)**.

- When Seema turns on her laptop, she must enter a **BitLocker PIN**. This unlocks the encryption key and allows access to the disk.
- After that, she logs in using her **Windows username and password**.
- If someone steals her laptop, the data remains protected unless the thief knows the BitLocker PIN.
- Even if the hard disk is removed and connected to another device, the contents will remain **ciphertext** (unreadable) unless the correct symmetric key is used.

**Key Point:**  
Symmetric encryption is **fast** and ideal for **encrypting large amounts of data**, like full disks. One key is used for both encryption and decryption.

---

### **Scenario 2: Symmetric Encryption Alone Isn't Enough for Internet Communication**  
![Alt text](/images/30d.png)
**Use Case:** Seema wants to access `pinkbank.com` securely  
**Challenge:** How to share a symmetric key securely?

Suppose Seema encrypts her online banking credentials using a symmetric key and sends them to `pinkbank.com`. Here's the issue:

- The bank **cannot decrypt the message** unless it has the same symmetric key.
- But Seema **cannot safely share** the key over the internet — an attacker could intercept it.

❌ **Problem:**  
Symmetric encryption **alone** is insecure for sending data over untrusted networks.

We'll revisit this in **Scenario 4**, where we solve this using **asymmetric encryption**.

---

### **Scenario 3: Asymmetric Encryption (SSH into EC2 Instance)**  
![Alt text](/images/30e.png)
**Use Case:** Secure login to Ubuntu EC2 instance on AWS  
**Technology Example:** SSH, Ed25519 keys

Seema wants to securely SSH into her Ubuntu EC2 instance. She uses **asymmetric encryption**, which involves a **public key** and a **private key**.

#### Steps:
1. **Generate key pair** using `ssh-keygen` (e.g., `mykey`, producing `mykey` and `mykey.key`).
2. **Copy the public key** (`mykey`) to the EC2 instance (usually placed in `~/.ssh/authorized_keys`).
3. **Set correct permissions** on the key files (especially for Linux systems).
4. **SSH into the EC2 instance** using the private key (`mykey.key`).

#### Key Concepts:
- If a message is **encrypted with Seema's public key**, **only her private key** can decrypt it.
- If something is **signed with her private key**, **anyone with the public key** can verify that it came from her.

#### How SSH Uses Asymmetric Encryption:
- The server sends a **random challenge**.
- Seema's SSH client **signs it locally** with her private key.
- The signed challenge is sent to the server.
- The server verifies it using her **public key**.
- If valid, she gets access — proving identity **without exposing** her private key.

**Key Point:**  
Asymmetric encryption is used for **identity verification and secure key exchange**. The **private key stays with the user**, and is **never transmitted**.

Here are some common extensions used for **public** and **private keys**:
- **Public Key Files**: `.pub`, `.crt`, `.cer`, `.pem`, `.der`
- **Private Key Files**: `.key`, `.pem`, `.ppk`, `.pfx`, `.der`

The most commonly used extensions for **SSH keys** are `.pub` for public keys and `.key` for private keys. For certificates in the SSL/TLS world, `.crt` and `.pem` are frequently used.

---

### **Scenario 4: Combining Symmetric + Asymmetric (TLS Handshake)** 
**Introducing TLS (Transport Layer Security) — The Foundation of Secure Web Communication**

Before we jump into Scenario 4, let’s understand **what TLS is and why it matters**.

- **TLS is the modern, more secure successor to SSL (Secure Sockets Layer)**. Although people still say "SSL certificates," what’s actually used today is TLS.
- TLS (Transport Layer Security) is the **security protocol** that powers **HTTPS**, ensuring data exchanged between your **browser and a website** is **private and secure**.
- It prevents attackers from **eavesdropping, tampering, or impersonating** the website you’re connecting to.
- TLS uses a combination of **asymmetric encryption** (for secure key exchange) and **symmetric encryption** (for fast and efficient data transfer).
- You’ve likely used TLS today — every time you saw a **lock icon in your browser**, TLS was working behind the scenes.

**TLS** is a **general-purpose protocol** used to secure various application-layer protocols like **HTTPS**, **FTPS**, **SMTPS**, **IMAPS/POP3S**, **LDAPs**, **gRPC**, **Kubernetes**, and **MQTT**. While **HTTPS** is the most common use of TLS, **TLS is not exclusive to HTTPS** and is used for secure communication in many other protocols. So, whenever you see **HTTPS**, you can be sure **TLS is working behind the scenes** to ensure **authentication**, **encryption**, and **integrity** of your communication.

Let’s now see how Seema’s browser establishes a secure TLS session with her bank’s website.

![Alt text](/images/30f.png)

**Use Case:** Seema accesses `pinkbank.com` securely via browser  
**Technology Example:** TLS (HTTPS), Digital Certificates, Certificate Authorities

Here’s what happens when Seema opens `https://pinkbank.com`:

1. **Seema’s browser requests `pinkbank.com`**
   - The browser says: “Hi, I’d like to start a secure conversation. Prove you’re really `pinkbank.com`.”

2. **`pinkbank.com` responds with its TLS certificate**
   This digital certificate contains:
   - The **public key** of `pinkbank.com`
   - The **domain name** it was issued for
   - A **digital signature from a Certificate Authority (CA)** like Let’s Encrypt

   > **Why a CA is needed**  
      A **Certificate Authority (CA)** is a trusted organization responsible for validating the ownership of domains and signing digital certificates.  
      When a domain owner (for example, the owner of `pinkbank.com`) registers their domain, they request a certificate from the CA. The CA performs a series of checks to ensure that the requester indeed controls the domain.  
      Once the verification is complete, the CA uses its **private key** to **sign the certificate**, creating a cryptographic endorsement of the domain's authenticity. This signature assures users that the domain is legitimate and that communication with it is secure.

    > **🔁 Link to Scenario 3**:  
      Remember in Scenario 3 (SSH login), **Seema's system used her private key to sign** a challenge sent by the server, and the **server verified it using the public key**.  
      Similarly here, in TLS:  
          - The **Certificate Authority (CA)** uses its **private key to sign** the TLS certificate.  
          - The **browser uses the CA’s public key** to **verify** the certificate’s authenticity.

3. **Browser verifies the certificate**
   - All browsers come with **a list of trusted CAs** and their **public keys**.
   - The browser checks:
     - Is the certificate signed by a trusted CA?
     - Has it expired?
     - Does the domain match (`pinkbank.com`)?
   - If all checks pass, the browser **trusts** that it's communicating with the real `pinkbank.com`.

4. **Browser generates a session key**
   - A random **symmetric session key** (e.g., for AES-256) is created.
   - This key will be used for encrypting all data sent afterward.

5. **Browser encrypts the session key using `pinkbank.com`’s public key**
   - Only `pinkbank.com`, with access to its **private key**, can decrypt this session key.

6. **`pinkbank.com` decrypts the session key**
   - It uses its private key to retrieve the session key.

7. **Secure communication begins**
   - Both browser and server now share the same symmetric session key.
   - All data is encrypted using this key — fast and secure.

---

### Client and Server – A Refresher

![Alt text](/images/31-1.png)

A **client** is the one that initiates a request; the **server** is the one that responds.

**General Examples:**
* When **Seema** accesses `pinkbank.com`, **Seema** is the **client**, and **pinkbank.com** is the **server**.
* When **Varun** downloads something from his **S3 bucket**, **Varun** is the **client**, and the **S3 bucket** is the **server**.

**Kubernetes Examples:**
* When **Seema** use `kubectl get pods`, `kubectl` is the **client** and the **API server** is the **server**.
* When the API server talks to `etcd`, the API server is now the **client**, and `etcd` is the **server**.

This direction of communication is critical when we later talk about **client certificates** and **mTLS**.

---
## **Public Key Cryptography**

### **Public Key Cryptography in DevOps: Focus on SSH & HTTPS**  

![Alt text](/images/31-2.png)

Public Key Cryptography (PKC) underpins authentication and secure communication across multiple **application-layer protocols**. While it's used in **email security (PGP, S/MIME), VoIP, database connections, and secure messaging**, a **DevOps engineer primarily interacts with SSH and HTTPS** for managing infrastructure.

Two essential tools for handling public-private key pairs in these domains are **ssh-keygen** (for SSH authentication) and **openssl** (for TLS certificates).

---

#### **1. Secure Remote Access: `ssh-keygen` (SSH Authentication)**  

`ssh-keygen` is the primary tool for generating SSH **public/private key pairs**, which enable **secure remote login** without passwords.  

- Used for **server administration, Git authentication, CI/CD pipelines, and automation**.  
- The **private key** is kept on the **client**, while the **public key** is stored on the **server** (`~/.ssh/authorized_keys`).  
- Authentication works via **public key cryptography**, where the server **verifies the client’s signed request** using its **stored public key**.  

**Alternative SSH Key Tools:**  
- **PuTTYgen** → Windows-based tool for generating SSH keys (used with PuTTY).  
- **OpenSSH** → Built-in on most Unix-based systems, provides SSH utilities including key management.  
- **Mosh (Mobile Shell)** → Used for remote connections and can leverage SSH key authentication.  

---


#### **2. Secure Web Communication: `openssl` (TLS Certificates & Identity Validation)**  

`openssl` is widely used for **private key generation** and **certificate management**, conforming to the **X.509 standard** for TLS encryption.  

- Generates a **private key**, which is securely stored on the server.  
- Creates a **certificate**, which contains a **public key**, metadata (issuer, validity), and a **digital signature from a Certificate Authority (CA)**.  
- The **private key** is used to establish secure communication, while the certificate allows clients to verify the server’s identity.  

> **Note:** TLS is not exclusive to HTTPS. Other application-layer protocols, such as **SMTP** (for email), **FTPS** (for secure file transfer), and **IMAPS** (for secure email retrieval), also use TLS for authentication and secure communication.

**Alternative TLS Certificate Tools:**

* **CFSSL (Cloudflare’s PKI Toolkit)** → Go-based toolkit by Cloudflare for generating and signing certificates.
* **HashiCorp Vault** → 	Can issue, sign, and manage certificates securely.
* **Certbot** → Automates key + cert generation from Let's Encrypt.

**Why TLS Still Uses a Tool Named "OpenSSL"**

At first glance, it may seem ironic that **TLS certificates are still generated using a tool called *OpenSSL***, especially since **SSL is obsolete** and has been replaced by TLS. But there's a historical and practical reason behind it.

**OpenSSL** originated as a toolkit to implement **SSL (Secure Sockets Layer)**, the predecessor to TLS. As SSL was deprecated and TLS became the standard, OpenSSL evolved to support all modern versions of **TLS** (including TLS 1.3), while keeping its original name for compatibility and familiarity.

But **OpenSSL is much more than just a certificate generator**. It’s a comprehensive **cryptographic toolkit** that can:

* Generate public/private key pairs
* Create and sign X.509 certificates
* Perform encryption, decryption, and hashing
* Handle PKI-related formats like PEM, DER, PKCS#12
* Test and debug secure sockets using TLS

So while the name "OpenSSL" may sound outdated, the tool itself is **modern, versatile, and still central** to secure communications today.

---

#### **Key Management Best Practices**  

To protect private keys from unauthorized access, consider these secure storage options:  

- **Hardware Security Modules (HSMs)** → Dedicated devices for key protection.  
- **Secure Enclaves (TPM, Apple Secure Enclave)** → Isolated hardware environments restricting key access.  
- **Cloud-based KMS (AWS KMS, Azure Key Vault)** → Encrypted storage with controlled access.  
- **Encrypted Key Files (`.pem`, `.pfx`)** → Secured with strong passwords.  
- **Smart Cards & USB Tokens (YubiKey, Nitrokey)** → Portable hardware-based security.  
- **Air-Gapped Systems** → Completely offline key storage to prevent network attacks.  

Regular **key rotation** and **audits** are crucial to maintaining security and replacing compromised keys efficiently. 

#### **Common Key File Formats**

#### **For SSH Authentication**

| Format                                | Description                                         |
| ------------------------------------- | --------------------------------------------------- |
| `.pub`                                | **Public key** (shared with the remote server)      |
| `.key`, `*-key.pem`, **no extension** | **Private key** (must be kept secure on the client) |




#### **For TLS Certificates**

| Format              | Description                                                     |
| ------------------- | --------------------------------------------------------------- |
| `.crt`, `.pem`      | **Certificate** (contains a public key and metadata)            |
| `.key`, `*-key.pem`, **no extension**  | **Private key** (must be securely stored)                       |
| `.csr`              | **Certificate Signing Request** (used to request a signed cert) |

By properly managing key storage and implementing best practices, organizations can significantly reduce security risks and prevent unauthorized access.

---

#### **What Public Key Cryptography (PKC) Provides**

Public Key Cryptography (PKC) underpins secure communication by enabling:
- Authentication (Identity Verification)
- Secure Key Exchange (Foundation for Encryption)

We will now discuss both of these in details.

### 1. **Authentication (Identity Verification):**

Public Key Cryptography enables entities—**users, servers, or systems**—to prove their identity using **key pairs** and **digital signatures**. This is fundamental to ensuring that communication is happening with a legitimate party.

#### **In SSH:**

![Alt text](/images/31-3.png)

SSH supports **mutual authentication**, where:

* **Server Authentication (Host Key):**

  * When an SSH server (e.g., a Linux machine) is installed, it automatically generates a **host key pair** for each supported algorithm (like RSA, ECDSA, ED25519), stored at:

    * **Private Key:** `/etc/ssh/ssh_host_<algo>_key`
    * **Public Key:** `/etc/ssh/ssh_host_<algo>_key.pub`

  * These keys are used to **prove the identity of the server** to any connecting SSH client.

  * When a client connects:

    * The server sends its **host public key**.

    * The SSH client checks whether this key is already present in its **`~/.ssh/known_hosts`** file.

  * If it's the **first connection**, the key won't exist in `known_hosts`, and the client will show a prompt:

    ```
    The authenticity of host 'server.com (192.168.1.10)' can't be established.
    ED25519 key fingerprint is SHA256:abc123...
    Are you sure you want to continue connecting (yes/no)?
    ```

    You can manually verify the fingerprint by running the following command **on the server** to print the fingerprint of its public host key:

    ```
    ssh-keygen -lf /etc/ssh/ssh_host_ed25519_key.pub
    ```

    * `-l` (lowercase L): shows the fingerprint of the key.
    * `-f`: specifies the path to the public key file.

    Replace `ssh_host_ed25519_key.pub` with the appropriate public key file (e.g., for RSA: `ssh_host_rsa_key.pub`) if using a different algorithm.


    * If the user accepts (`yes`), the server's host key is **saved in `~/.ssh/known_hosts`** and used for verification in all future connections.

    * This approach is called **TOFU (Trust On First Use)**—you trust the server the first time, and ensure its identity doesn't change later.
    * Host Key Verification Methods:

      1. **TOFU (Trust On First Use)**
        Accept the host key manually when prompted during the first connection. Risky on untrusted networks.

      2. **Phone-a-Friend**
        Contact someone who already has access to the server and confirm the server's fingerprint via a secure channel.

      3. **OOB (Out-of-Band) Verification**
        Compare the fingerprint out-of-band (e.g., via Slack, secure email, a trusted documentation portal, etc.).

      4. **Ansible or Configuration Management**
        Use automation tools like Ansible, Puppet, or Chef to push known host fingerprints into `~/.ssh/known_hosts`.

      5. **Pre-loading Known Hosts**
        Populate the `~/.ssh/known_hosts` or `/etc/ssh/ssh_known_hosts` file manually with the correct public key before connecting:

        ```bash
        ssh-keyscan server.example.com >> ~/.ssh/known_hosts
        ```

      6. **Centralized Trust Models**
        In enterprises, SSH certificates (using OpenSSH's CA support) can be used where a trusted internal CA signs host keys and clients trust the CA.

  * On subsequent connections:

    * If the server's host key has changed (possibly due to a reinstallation or a **man-in-the-middle attack**), SSH will warn the user and may block the connection unless the mismatch is explicitly resolved.

    * This warning may also appear in **cloud environments** where the server’s **ephemeral public IP** changes — even if the host key is the same — because SSH uses the IP/hostname to identify the server.

* **Client Authentication:**

  * The **server sends a challenge**, and the **client must sign it using its private key** (e.g., `~/.ssh/mykey`) to prove it possesses the corresponding private key.
  * The **server verifies the signature** using the **client’s public key**, which must be present in the server’s `~/.ssh/authorized_keys` file.

#### **In TLS (e.g., HTTPS):**

* The server presents a **digital certificate** (issued and signed by a **trusted Certificate Authority**) to the client.
* This certificate includes the server’s **public key** and a **CA signature**.
* The client verifies:
  * That the certificate is **issued by a trusted CA** (whose root cert is pre-installed).
  * That the certificate matches the **domain name** (e.g., `example.com`).
* Optionally, in **mutual TLS**, the client also presents a certificate for authentication.

---

### Order of Authentication in Mutual Authentication (SSH or mTLS)

Whether using **SSH** or **mutual TLS (mTLS)**, the **server always authenticates first**, followed by the client.

**Reasons the server authenticates first:**

* The client must verify the server's identity before sending any sensitive data.
* It prevents man-in-the-middle (MITM) attacks by ensuring the connection is to the intended endpoint.
* A secure, encrypted channel is established only after the server is trusted.
* This pattern ensures that secrets (like private keys or credentials) are never exposed to an untrusted server.

This sequence holds true for:

* SSH (server presents host key first, then client authentication occurs)
* mTLS (server presents its certificate first, then requests the client's certificate if required)

---

### 2. **Secure Key Exchange (Foundation for Encryption):**

Rather than encrypting all data directly using asymmetric cryptography, **Public Key Cryptography (PKC)** is used to **securely exchange or derive symmetric session keys**. These **session keys** are then used for efficient **symmetric encryption** of actual data in transit.

* In **SSH**:

  * Both the **client and server participate equally** in a **key exchange algorithm** (e.g., **Diffie-Hellman** or **ECDH (Elliptic Curve Diffie-Hellman)**).
  * Each side contributes a random component and uses the other’s public part to **jointly compute the same session key**.
  * **No single party creates the session key outright**; it is derived **collaboratively**, ensuring that **neither side sends the session key directly** — making it secure even over untrusted networks.
  * >🔐 **Importantly, this session key is established and encryption begins *before* any client authentication takes place.**
    This ensures that even authentication credentials (like passwords or signed challenges) are never sent in plaintext.

* In **TLS**:

  * **Older TLS (e.g., TLS 1.2 with RSA key exchange)**
![Alt text](/images/31-8.png)
    * The client generates a session key.
    * It encrypts the session key using the server’s public key (from its certificate).
    * The server decrypts it using its private key.
    * Both sides use this session key for symmetric encryption.
      **Drawback**: If the server’s private key is compromised, past sessions can be decrypted (no forward secrecy).

  * **Modern TLS (e.g., TLS 1.3 or TLS 1.2 with ECDHE)**
![Alt text](/images/31-4.png)
    * The client and server perform an ephemeral key exchange (e.g., ECDHE – Elliptic Curve Diffie-Hellman Ephemeral).
    * Both sides derive the session key collaboratively — it is never transmitted directly.
      **Advantage**: Even if the server’s private key is compromised later, past communications remain secure (forward secrecy).

> * **In SSH, encryption begins *before* authentication**, ensuring that even credentials are exchanged over a secure channel.
>* **In HTTPS (TLS), authentication happens *before* encryption**, as the server must first prove its identity via certificate.

---

### Types of TLS Certificate Authorities (CA): Public, Private, and Self-Signed

When enabling HTTPS or TLS for applications, certificates must be signed to be trusted by clients. There are three common ways to achieve this:
1. **Public CA** – Used for production websites accessible over the internet (e.g., Let's Encrypt, DigiCert).
2. **Private CA** – Used within organizations for internal services (e.g., `*.internal` domains).
3. **Self-Signed Certificates** – Quick to create, mainly used for testing, but not trusted by browsers.

**Public CA vs Private CA vs Self-Signed Certificates**

| **Certificate Type**         | **Use Case**                                      | **Trust Level**                 | **Common Examples**                       | **Typical Use**                                                                        |
| ---------------------------- | ------------------------------------------------- | ------------------------------- | ----------------------------------------- | -------------------------------------------------------------------------------------- |
| **Public CA**                | Production websites, accessible over the internet | Trusted by all major browsers   | Let's Encrypt, DigiCert, GlobalSign       | Used for production environments and public-facing sites                               |
| **Private CA**               | Internal services within an organization          | Trusted within the organization | Custom CA (e.g., internal enterprise CAs) | Used for internal applications, such as `*.internal` domains                           |
| **Self-Signed Certificates** | Testing and development                           | Not trusted by browsers         | N/A                                       | Quick certificates for testing or development purposes, not recommended for production |

---

### Public CA

![Alt text](/images/31-5.png)

When you visit a website like `pinkbank.com`, your browser needs a way to verify that the server it’s talking to is indeed `pinkbank.com` and not someone pretending to be it. That’s where a **Certificate Authority (CA)** comes into play.

* `pinkbank.com` generates its own digital certificate (often through a **Certificate Signing Request** using tools like OpenSSL) and then gets it signed by a trusted Certificate Authority (CA) such as Let’s Encrypt to prove its authenticity.
* Seema’s browser, like most browsers, already contains the **public keys of well-known CAs**. So, it can verify that the certificate presented by `pinkbank.com` is indeed signed by Let’s Encrypt.
* This trust chain ensures authenticity. Without a CA, there would be no trusted way to confirm the server’s identity.

If a certificate were self-signed or signed by an unknown entity, Seema’s browser would show a warning because it cannot validate the certificate's authenticity.

**Important Note**
In this example, I used Let’s Encrypt because it is a popular choice for DevOps engineers, developers, and cloud engineers, as it provides free, automated SSL/TLS certificates.
> While Let’s Encrypt is widely used even in production environments, especially for public-facing services, enterprise use cases may also involve certificates from commercial Certificate Authorities (CAs) like DigiCert, GlobalSign, Entrust, or Google Trust Services — which offer advanced features like extended validation (EV), organization validation (OV), SLAs, and dedicated support.

---

**Public Key Infrastructure (PKI)**

PKI is a framework that manages digital certificates, keys, and Certificate Signing Requests (CSRs) to enable secure communication over networks. It involves the use of **public and private keys** to encrypt and decrypt data, ensuring confidentiality and authentication.

---

### Private CA

![Alt text](/images/31-6.png)

Just like browsers come with a list of trusted CAs, you can manually add a CA’s public key to your trust store (e.g., in a browser or an operating system).

**Summary for Internal HTTPS Access Without Warnings**

To securely expose an internal app as `https://app1.internal` without browser warnings:

* Set up a **private Certificate Authority (CA)** and issue a TLS certificate for `app1.internal`.

* Install the **private CA’s root certificate** on all internal user machines so their browsers trust the certificate:

  * **Windows**: Use **Group Policy (GPO)** to add the CA cert to the **Trusted Root Certification Authorities** store.
  * **macOS**: Use **MDM** or manually import the root cert using **Keychain Access** → System → Certificates → Trust.
  * **Linux**: Place the CA cert in `/usr/local/share/ca-certificates/` and run:

    ```bash
    sudo update-ca-certificates
    ```

* Ensure internal DNS resolves `app1.internal` to the correct internal IP.

---

### Self-Signed Certificate

![Alt text](/images/31-7.png)

A **self-signed certificate** is a certificate that is **signed with its own private key**, rather than being issued by a trusted Certificate Authority (CA).

Let’s take an example:
Our developer **Shwetangi** is building an internal application named **app2**, accessible locally at **app2.test**. She wants to enable **HTTPS** to test how her application behaves over a secure connection. Since it's only for development, she generates a self-signed certificate using tools like `openssl` and uses it to enable HTTPS on **app2.test**.

#### **Typical Use Cases of Self-Signed Certificates**

* Local development and testing environments
* Internal tools not exposed publicly
* Quick prototyping or sandbox setups
* Lab or non-production Kubernetes clusters

> ⚠️ Self-signed certificates are **not trusted by browsers or clients** by default and will trigger warnings like:
> Chrome: “Your connection is not private” (NET::ERR\_CERT\_AUTHORITY\_INVALID)
> Firefox: “Warning: Potential Security Risk Ahead”

#### **Common Internal Domain Suffixes for Testing**

* `.test` — Reserved for testing and documentation (RFC 6761)
* `.local` — Often used by mDNS/Bonjour or local network devices
* `.internal` — Used in private networks or cloud-native environments (e.g., GCP)
* `.dev`, `.example` — Reserved for documentation and sometimes local use

Using these reserved domains helps avoid accidental DNS resolution on the public internet and is a best practice for local/dev setups.

---

## Kubeconfig and Kubernetes Context

### What is a Kubeconfig File and Why Do We Need It?

![Alt text](/images/31-9.png)

* The **kubeconfig file** is a configuration file that stores:

  * **Cluster connection information** (e.g., API server URL)
  * **User credentials** (e.g., authentication tokens, certificates)
  * **Context information** (e.g., which cluster and user to use)
* It allows Kubernetes tools (like `kubectl`) to interact with the cluster securely and seamlessly.
* The kubeconfig file simplifies authentication and access control, eliminating the need to manually provide connection details each time we run a command.

### What If the Kubeconfig File Were Not There?

Without the kubeconfig file, you would need to manually specify the cluster connection information for every command. For example:

* With the kubeconfig file:

  ```bash
  kubectl get pods
  ```

* Without the kubeconfig file, you would need to include details like this:

  ```bash
  kubectl get pods \
    --server=https://<API_SERVER_URL> \
    --certificate-authority=<CA_CERT_FILE> \
    --client-key=<CLIENT_KEY_FILE> \
    --client-certificate=<CLIENT_CERT_FILE>
  ```

As you can see, without the kubeconfig file, the command becomes longer and more error-prone. Each command requires explicit details, which can be tedious to manage.

**We will look into how a kubeconfile looks like in a while.**

---

### Kubernetes Context
**Kubernetes contexts** allow users to easily manage multiple clusters and namespaces by storing cluster, user, and namespace information in the `kubeconfig` file. Each context defines a combination of a cluster, a user, and a namespace, making it simple for users like `Seema` to switch between clusters and namespaces seamlessly without manually changing the configuration each time.

![Alt text](/images/31-10.png)

## Scenario:

Seema, the user, wants to access three different Kubernetes clusters from her laptop. Let's assume these clusters are:

1. **dev-cluster** (for development)
2. **staging-cluster** (for staging)
3. **prod-cluster** (for production)

Seema will need to interact with these clusters frequently. Using Kubernetes contexts, she can set up and switch between these clusters easily.

## How Kubernetes Contexts Work:

### 1. Configuring Contexts:

In the `kubeconfig` file, each cluster, user, and namespace combination is stored as a context. So, Seema can have three different contexts, one for each cluster. The contexts will contain:

* **Cluster details**: API server URL, certificate authority, etc.
* **User details**: Authentication method (e.g., username/password, token, certificate).
* **Namespace details**: The default namespace to work in for the context (though the namespace can be overridden on a per-command basis).

### 2. Switching Between Contexts:

With Kubernetes contexts configured, Seema can easily switch between them. If she needs to work on **dev-cluster**, she can switch to the `dev-context`. Similarly, for **staging-cluster**, she switches to `staging-context`, and for **prod-cluster**, the `prod-context`.

A **Kubernetes namespace** is a virtual cluster within a physical cluster that provides logical segregation for resources. This allows multiple environments like dev, staging, and prod to coexist on the same cluster without interference.

We now use naming like `app1-dev-ns`, `app1-staging-ns`, and `app1-prod-ns` to logically group and isolate resources of `app1` across environments.

For example, `app1-dev-ns` in dev-cluster keeps all resources related to `app1`'s development isolated from production resources in `app1-prod-ns` on prod-cluster.

## Example `kubeconfig` File:

> 🗂️ **Note:** The `kubeconfig` file is usually located at `~/.kube/config` in the home directory of the user who installed `kubectl`. This file allows users to connect to one or more clusters by managing credentials, clusters, and contexts. While the file is commonly named `config`, it's often referred to as the *kubeconfig* file, and its location can vary depending on the cluster setup.

![Alt text](/images/31-12.png)

Here's a simplified example of what the `kubeconfig` file might look like:

```yaml
apiVersion: v1
# Define the clusters
clusters:
  - name: dev-cluster  # Logical name for the development cluster
    cluster:
      server: https://dev-cluster-api-server:6443  # API server endpoint (usually port 6443)
      certificate-authority-data: <certificate-data>  # Base64-encoded CA certificate
  - name: staging-cluster
    cluster:
      server: https://staging-cluster-api-server:6443
      certificate-authority-data: <certificate-data>
  - name: prod-cluster
    cluster:
      server: https://prod-cluster-api-server:6443
      certificate-authority-data: <certificate-data>

# Define users (credentials for authentication)
users:
  - name: seema  # Logical user name
    user:
      client-certificate-data: <client-cert-data>  # Base64-encoded client certificate
      client-key-data: <client-key-data>  # Base64-encoded private key

# Define contexts (combination of cluster + user + optional namespace)
contexts:
  - name: seema@dev-cluster-context
    context:
      cluster: dev-cluster
      user: seema
      namespace: app1-dev-ns  # Default namespace for this context; kubectl commands will run in this namespace when this context is active
  - name: seema@staging-cluster-context
    context:
      cluster: staging-cluster
      user: seema
      namespace: app1-staging-ns  # When this context is active, kubectl will run commands in this namespace
  - name: seema@prod-cluster-context
    context:
      cluster: prod-cluster
      user: seema
      namespace: app1-prod-ns

# Set the default context to use
current-context: seema@dev-cluster-context

# -------------------------------------------------------------------
# Explanation of Certificate Fields:

# certificate-authority-data:
#   - This is the base64-encoded public certificate of the cluster’s Certificate Authority (CA).
#   - Used by the client (kubectl) to verify the identity of the API server (ensures it is trusted).

# client-certificate-data:
#   - This is the base64-encoded public certificate issued to the user (client).
#   - Sent to the API server to authenticate the user's identity.

# client-key-data:
#   - This is the base64-encoded private key that pairs with the client certificate.
#   - Used to prove the user's identity securely to the API server.
#   - Must be kept safe, as it can be used to impersonate the user.

# Together, these enable secure mutual TLS authentication between kubectl and the Kubernetes API server.

```
---

### Viewing & Managing Your `kubeconfig`

Your `kubeconfig` file (typically at `~/.kube/config`) holds info about **clusters, users, contexts, and namespaces**. Avoid editing it manually—use `kubectl config` for safe, consistent changes.

> **Pro Tip:** Use `kubectl config -h` to explore powerful subcommands like `use-context`, `set-context`, `rename-context`, etc.

---

### Common `kubectl config` Commands (Compact View)

| Task                                      | Command & Example                                                                                                                                                                        |
| ----------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| View current context                      | `kubectl config current-context`<br>Example: `seema@dev-cluster-context`                                                                                                                 |
| List all contexts                         | `kubectl config get-contexts`<br>Shows: `seema@dev-cluster-context`, etc.                                                                                                                |
| Switch to a different context             | `kubectl config use-context seema@prod-cluster-context`<br>Switches to prod                                                                                                              |
| View entire config (pretty format)        | `kubectl config view`<br>Readable view of clusters, users, contexts                                                                                                                      |
| View raw config (YAML for scripting)      | `kubectl config view --raw`<br>Useful for parsing/exporting                                                                                                                              |
| Show active config file path              | `echo $KUBECONFIG`<br>Defaults to `~/.kube/config` if not explicitly set                                                                                                                 |
| Set default namespace for current context | `kubectl config set-context --current --namespace=app1-staging-ns`<br>Updates Seema's current context                                                                                    |
| Override namespace just for one command   | `kubectl get pods --namespace=app1-prod-ns`<br>Runs the command in prod namespace                                                                                                        |
| Inspect kubeconfig at custom path         | `kubectl config view --kubeconfig=~/kubeconfigs/custom-kubeconfig.yaml`                                                                                                                  |
| Add a new user                            | `kubectl config set-credentials varun --client-certificate=varun-cert.pem --client-key=varun-key.pem --kubeconfig=~/kubeconfigs/seema-kubeconfig.yaml`                                   |
| Add a new cluster                         | `kubectl config set-cluster dev-cluster --server=https://dev-cluster-api-server:6443 --certificate-authority=ca.crt --embed-certs=true --kubeconfig=~/kubeconfigs/seema-kubeconfig.yaml` |
| Add a new context                         | `kubectl config set-context seema@dev-cluster-context --cluster=dev-cluster --user=seema --namespace=app1-dev-ns --kubeconfig=~/kubeconfigs/seema-kubeconfig.yaml`                       |
| Rename a context                          | `kubectl config rename-context seema@dev-cluster-context seema@dev-env`                                                                                                                  |
| Delete a context                          | `kubectl config delete-context seema@staging-cluster-context`                                                                                                                            |
| Delete a user                             | `kubectl config unset users.seema`                                                                                                                                                       |
| Delete a cluster                          | `kubectl config unset clusters.staging-cluster`                                                                                                                                          |
---

### Multiple Kubeconfig Files

Kubernetes supports multiple config files. Use `--kubeconfig` with any `kubectl` command to specify which one to use:

```bash
kubectl get pods --kubeconfig=~/.kube/dev-kubeconfig
kubectl config use-context dev-cluster --kubeconfig=~/.kube/dev-kubeconfig
```

Ideal for managing multiple clusters (e.g., dev/staging/prod) cleanly.

---

### Using a Custom Kubeconfig as Default via `KUBECONFIG`

By default, `kubectl` uses the kubeconfig file at `~/.kube/config`. You won't see anything with `echo $KUBECONFIG` unless you've set it yourself.

To avoid specifying `--kubeconfig` with every command, you can set the `KUBECONFIG` environment variable:

**Step-by-step:**

1. Open your shell profile:

   ```bash
   vi ~/.bashrc   # Or ~/.zshrc, depending on your shell
   ```

2. Add the line:

   ```bash
   export KUBECONFIG=$HOME/.kube/my-2nd-kubeconfig-file
   ```

3. Apply the change:

   ```bash
   source ~/.bashrc
   ```

Now, `kubectl` will automatically use that file for all commands.

---

### Adding Entries to a Kubeconfig File

Avoid manually editing the kubeconfig. Instead, use `kubectl config` to manage users, clusters, and contexts.

**Add a new user:**

```bash
kubectl config set-credentials varun \
  --client-certificate=~/kubeconfigs/varun-cert.pem \
  --client-key=~/kubeconfigs/varun-key.pem \
  --kubeconfig=~/kubeconfigs/my-2nd-kubeconfig-file
```

**Add a new cluster:**

```bash
kubectl config set-cluster aws-cluster \
  --server=https://aws-api-server:6443 \
  --certificate-authority=~/kubeconfigs/aws-ca.crt \
  --embed-certs=true \
  --kubeconfig=~/kubeconfigs/my-2nd-kubeconfig-file
```

**Add a new context:**

```bash
kubectl config set-context varun@aws-cluster-context \
  --cluster=aws-cluster \
  --user=varun \
  --namespace=default \
  --kubeconfig=~/kubeconfigs/my-2nd-kubeconfig-file
```

**Switch to the new context:**

```bash
kubectl config use-context varun@aws-cluster-context \
  --kubeconfig=~/kubeconfigs/my-2nd-kubeconfig-file
```
**Verify:**



```bash
kubectl config view --kubeconfig=~/kubeconfigs/my-2nd-kubeconfig-file --minify
```
 * `--kubeconfig=...`: Points to your custom kubeconfig file.
 * `--minify`: Shows only the active context and related cluster/user info.
 
```bash
kubectl config view --kubeconfig=~/kubeconfigs/my-2nd-kubeconfig-file
```



---

## Mutual TLS (mTLS)

**Mutual TLS (mTLS)** is an extension of standard TLS where **both the client and server authenticate each other** using digital certificates. This ensures that **each party is who they claim to be**, providing strong identity verification and preventing unauthorized access. mTLS is especially important in environments where **machines, services, or workloads** need to communicate securely without human involvement — such as in microservices, Kubernetes clusters, and service meshes.

We often observe that **public-facing websites**, typically accessed by **humans via browsers**, use **one-way TLS**, where **only the server is authenticated**. For example, when Seema visits `pinkbank.com`, the server proves its identity by presenting a **signed certificate** issued by a **trusted Certificate Authority (CA)**.

However, in **machine-to-machine communication** — such as in **Kubernetes** or other distributed systems — it's common to see **mutual TLS (mTLS)**. In this model, **both parties present certificates**, enabling **bi-directional trust** and stronger security.

All major **managed Kubernetes services**, including **Amazon EKS**, **Google GKE**, and **Azure AKS**, **enforce mTLS by default** for internal control plane communication (e.g., between the API server, kubelet, controller manager, and etcd), as part of their secure-by-default approach.

> Whether it's **SSH** or **TLS/mTLS**, the **server always proves its identity first**. This is by design — the client must be sure it is talking to the correct, trusted server **before** sending any sensitive data or credentials.

---

**Why mTLS in Kubernetes?**

* Prevents unauthorized components from communicating within the cluster.
* Ensures that only trusted services (e.g., a valid API server or kubelet) can connect to each other.
* Strengthens the overall security posture of the cluster, especially in production environments.

Although some components can work with just server-side TLS, enabling mTLS is **strongly recommended** wherever possible — particularly in communication with sensitive components like `etcd`, `kubelet`, and `kube-apiserver`.

| Feature               | TLS (1-Way)             | mTLS (2-Way)                               |
| --------------------- | ----------------------- | ------------------------------------------ |
| Authentication        | Server only             | **Both client and server**                 |
| Use Case              | Public websites, APIs   | Microservices, Kubernetes, APIs w/ clients |
| Identity Verification | Server proves identity  | **Mutual identity verification**           |
| Certificate Required  | Only server certificate | **Both server and client certificates**    |
| Security Level        | Strong                  | **Stronger (mutual trust)**                |

---

### **Private CAs in Kubernetes Clusters**

![Alt text](/images/33b.png)

In Kubernetes, **TLS certificates secure communication** between core components. These certificates are signed by a **private Certificate Authority (CA)**, ensuring authentication and encryption. Depending on security requirements, a cluster can be configured to use:

* A **single CA** for the entire cluster (simpler, easier to manage), or
* **Multiple CAs** (e.g., a separate CA for etcd) for added security and isolation.

**Why Use Multiple CAs?**

Using **multiple private CAs** strengthens security by **isolating trust boundaries**. This approach is particularly useful for sensitive components like **etcd**, which stores the **entire cluster state**.

Security Considerations:
- If **etcd is compromised**, an attacker could gain full control over the cluster.
- To **limit the blast radius** in case of a CA or key compromise, it's common to **assign a separate CA exclusively for etcd**.
- Other control plane components (e.g., API server, scheduler, controller-manager) can share a **different CA**, ensuring **compartmentalized trust**.

By segmenting certificate authorities, Kubernetes operators can **reduce risk** and **enhance security posture**.

You can configure Kubernetes components to **trust a private CA** by distributing the CA’s public certificate to each component’s trust store. For example:

* `kubectl` trusts the API server because it has the **private CA** certificate that signed the API server's certificate.
* Similarly, components like the `controller-manager`, `scheduler`, and `kubelet` trust the API server and each other using certificates signed by this **shared private CA**.

Most Kubernetes clusters—whether provisioned via tools like `kubeadm`, `k3s`, or through managed services like **EKS**, **GKE**, or **AKS**—automatically generate and manage a **private CA** during cluster initialization. This CA is used to issue certificates for key components, enabling **TLS encryption and mutual authentication** out of the box.

When using **managed Kubernetes services**, this private CA is maintained by the cloud provider. It remains **hidden from users**, but all internal components are configured to trust it, ensuring secure communication without manual intervention.

However, when setting up a cluster **“the hard way”** (e.g., via [Kelsey Hightower’s guide](https://github.com/kelseyhightower/kubernetes-the-hard-way)), **you are responsible for creating and managing the entire certificate chain**. This means:

* Generating a root CA certificate and key,
* Signing individual component certificates,
* And distributing them appropriately.

While this approach offers **maximum transparency and control**, it also demands a solid understanding of **PKI, TLS, and Kubernetes internals**.

---

**Do Enterprises Use Public CAs for Kubernetes?**

Enterprises do **not** use public Certificate Authorities (CAs) for **core Kubernetes internals**. Instead, they rely on **private CAs**—either auto-generated (using tools like `kubeadm`) or centrally managed—to sign certificates used by Kubernetes components like the API server, kubelet, controller-manager, and scheduler. These certificates facilitate secure **TLS encryption and mutual authentication** within the cluster.

For **public-facing services**, however, it's common to use **public CAs**. Components such as:

* Ingress controllers
* Load balancers
* Gateway API implementations

...require certificates trusted by browsers. In these cases, enterprises use public CAs (e.g., Let’s Encrypt, DigiCert, GlobalSign) to issue TLS certificates, ensuring a secure **HTTPS** experience for users and avoiding browser trust warnings.

In short:

* **Private CAs** → Used for internal Kubernetes communication
* **Public CAs** → Used for securing external-facing applications
* ⚠️ Public CAs are **not** used for control plane or internal Kubernetes components
---

### Kubernetes Components as Clients and Servers

In the diagram below, arrows represent the direction of client-server communication:

![Alt text](/images/33a.png)

* The **arrow tail** indicates the **client**, and the **arrowhead** points to the **server**.
* Some arrows have **only arrowheads** (e.g., between **kubelet** and **API server**) to indicate that **the server initiates the connection** in specific cases:

  * When a user runs commands like `kubectl logs` or `kubectl exec`, the **API server acts as the client**, reaching out to the **kubelet**.
  * Conversely, when the **kubelet pushes node or pod health data**, it becomes the **client**, and the **API server** is the **server**.
* The **etcd arrow is colored yellow** to indicate that **etcd always acts as a server**, receiving requests from the API server.

---

### **Client (Initiates a Request)**

1. **kubectl**: **Interacts with the API server**, used by admins and DevOps engineers for cluster management, deployment, viewing logs.
2. **Scheduler**: **Requests the API server** for pod scheduling, checks for unscheduled pods, manages resource placement.
3. **API Server**: **Communicates with etcd**, **interacts with kubelet** for logs, exec commands, etc.
4. **Controller Manager**: **Requests the API server** to verify desired vs. current state, manages controllers, ensures cluster state matches desired configuration.
5. **Kube-Proxy**: **Communicates with the API server** for service discovery and endpoints, acts as a network router for traffic.
6. **Kubelet**: **Reports to the API server** about node health and pod status, fetches ConfigMaps & Secrets, ensures desired containers are running.

---
### **Server (Responds to the Request)**

1. **API Server**: **Responds to kubectl**, admins, DevOps, and third-party clients, manages resources and cluster state.
2. **etcd**: **Responds to API server**, stores cluster configuration, state, and secrets (only the API server interacts with etcd).
3. **Kubelet**: **Responds to API server** for pod status, logs, and exec commands.

> The roles mentioned above for clients and servers are **indicative**, not exhaustive. Kubernetes components may interact in multiple ways, and their responsibilities can evolve as the system grows and new features are added.

**Client or Server? It Depends on the Context**

In Kubernetes, whether a component acts as a **client** or **server** depends entirely on the direction of the request.

A single component can play both roles depending on the scenario. For example, when a user or a tool like `kubectl` accesses the **API server**, the API server acts as the **server**. However, when the **API server** communicates with another component like the **kubelet**—for fetching logs (`kubectl logs`), executing (`kubectl exec`) into containers, or retrieving node and pod status—the API server becomes the **client**, and the kubelet acts as the **server**.

Components such as the **scheduler**, **controller manager**, and **kube-proxy** are always **clients** because they initiate communication with the **API server** to get the desired cluster state, pod placements, or service endpoints.

On the other hand, **etcd** is **always a server** in the Kubernetes architecture. It **only** communicates with the **API server**, which acts as its **client** — no other component talks to etcd directly. This design keeps etcd isolated and secure, as it holds the cluster’s source of truth.

> **Note:** In **HA etcd setups**, each etcd node also acts as a **client** when talking to its **peer members** for cluster replication and consensus.
> However, this internal etcd-to-etcd communication is **independent of the Kubernetes control plane**.

---

## Understanding TLS in Kubernetes: No More Memorizing File Paths

TLS and certificates are core to how Kubernetes secures communication between its components — but let’s be honest, most explanations out there reduce it to a bunch of file paths and flags you’re expected to memorize.

We’re going to take a different approach.

Instead of rote learning, you'll learn how to **investigate and reason through** TLS communications in Kubernetes by **reading and interpreting configuration files**. This way, no matter the cluster setup — whether it’s a managed cloud offering or a custom on-prem deployment — you’ll know exactly where to look and how to figure things out.

**What You’ll Learn from the Next 3 Examples**

We’ll walk through **three real-world scenarios** of **mutual TLS (mTLS)** between core Kubernetes components. In each one, you’ll learn how to:

* **Identify who is the client and who is the server**
* **Determine which certificate is presented by each party**
* **Understand which CA signs those certificates and how trust is established**
* **Trace certificate locations and verify them using OpenSSL**
* **Inspect kubeconfig files to understand client behavior**

> **Key Idea:** You do **not** need to memorize certificate paths.
> You just need to know **where to look** — and that’s always the component’s configuration file.


**Why These Examples?**

These are not hand-picked just for simplicity. The interactions between:

1. **Scheduler and API Server**
2. **API Server and etcd**
3. **API Server and Kubelet**

…cover the **most critical and foundational TLS flows** in Kubernetes. If you understand these, you can apply the same logic to almost any other TLS interaction in the cluster — including kubelet as a client, webhooks, and even ingress traffic.

---

## Example 1: mTLS Between Scheduler (Client) and API Server (Server)

![Alt text](/images/33c.png)

Let’s walk through an end-to-end explanation of how **mutual TLS (mTLS)** works in Kubernetes — specifically, the communication between the **kube-scheduler** (client) and the **kube-apiserver** (server).

This example will show you **how to figure out certificate paths and their roles** by inspecting configuration files — **not by memorizing paths**.

---

### Key Principle: All Clients Use a Kubeconfig File to Connect to the API Server

> **IMPORTANT:** Anything that connects to the API server — whether it's `kubectl`, a control plane component like the scheduler or controller-manager, or an automation tool — uses a **kubeconfig file** to do so.

We already saw this in action in a previous lecture when **Seema**, using `kubectl`, connected to the API server using her personal `~/.kube/config` file. The same logic applies here: the **scheduler is also a client**, and its communication with the API server is facilitated by a kubeconfig file — in this case, `/etc/kubernetes/scheduler.conf`.

> 🔍 For `kube-proxy`, since it runs as a **DaemonSet**, its kubeconfig file is mounted into each pod. You can `kubectl exec` into a `kube-proxy` pod and find the kubeconfig typically at:
> `/var/lib/kube-proxy/kubeconfig`

This principle not only helps you understand **how TLS authentication is wired** in Kubernetes, but also gives you a **consistent mental model** for troubleshooting and inspecting certificates.

---

## Key TLS and Configuration File Locations (Control Plane & Nodes)

| Purpose                                  | Typical Path                                            | Notes                                                                  |
| ---------------------------------------- | ------------------------------------------------------- | ---------------------------------------------------------------------- |
| Kubeconfig files                         | `/etc/kubernetes/*.conf`                                | Used by core components (e.g., controller-manager, scheduler, kubelet) |
| Static Pod manifests                     | `/etc/kubernetes/manifests`                             | Includes API server, controller-manager, etcd,  scheduler                     |
| API server & control plane certs         | `/etc/kubernetes/pki`                                   | Certificates and keys for API server                                   |
| etcd certificates                        | `/etc/kubernetes/pki/etcd`                              | Only present if etcd is running locally                                |
| Kubelet certificates (on **all nodes**)  | `/var/lib/kubelet/pki`                                  | Includes kubelet server & client certs                                 |
| kube-proxy configuration (DaemonSet pod) | Inside `/var/lib/kube-proxy/` or mounted into container | Use `kubectl exec` into the kube-proxy pod to inspect                  |

---

## SERVER-SIDE: How the API Server Presents Its Certificate

**Goal:** Understand what certificate the API server presents and how the scheduler verifies it.

---

### Step 1: Who’s the Server?

In this interaction:

* **Scheduler** → acts as the **client**
* **API Server** → acts as the **server**

---

### Step 2: How Does the Server Present a Certificate?

Since API server is a static pod, inspect its manifest:

```bash
cat /etc/kubernetes/manifests/kube-apiserver.yaml
```

Look for this flag:

```yaml
--tls-cert-file=/etc/kubernetes/pki/apiserver.crt
```

This tells us the API server presents this certificate:

```bash
/etc/kubernetes/pki/apiserver.crt
```

To view its contents:

```bash
openssl x509 -in /etc/kubernetes/pki/apiserver.crt -text -noout
```

You’ll see output like:

```
Subject: CN=kube-apiserver
Issuer: CN=kubernetes
```

This confirms:

* The API server identifies itself as `kube-apiserver`
* The certificate is signed by the **Kubernetes cluster CA**

---

### Step 3: How Does the Scheduler Trust the API Server’s Certificate?

To verify the server's certificate, the **scheduler must trust the CA** that signed it.

Here's how that works:

#### a. Locate the Kubeconfig File

The scheduler uses a kubeconfig to connect to the API server. Find it by checking:

```bash
cat /etc/kubernetes/manifests/kube-scheduler.yaml
```

Look for:

```yaml
--kubeconfig=/etc/kubernetes/scheduler.conf
```

#### b. Inspect the CA Data

Open the kubeconfig file:

```bash
cat /etc/kubernetes/scheduler.conf
```

Look for:

```yaml
certificate-authority-data: <base64 encoded cert>
```

This is the **CA certificate** (in base64) used to validate the API server’s cert.

You can decode and inspect it:

```bash
echo -n "<base64 data>" | base64 --decode > ca.crt
openssl x509 -in ca.crt -text -noout
```

This certificate will show:

```
Subject: CN=kubernetes
Issuer: CN=kubernetes
```

That means:

* It’s a **self-signed cluster root CA**
* The **scheduler trusts this CA**
* So it can **verify the identity of the API server**

---

**Why Do Popular Certificate Authorities Also Have a Root CA?**
  Many well-known Certificate Authorities (CAs), like **Sectigo**, **Let's Encrypt**, or **DigiCert**, operate their own **Root Certificate Authority (Root CA)** in addition to one or more **Intermediate CAs**.

  **What Is a Root CA and Why Does It Exist?**
  * A **Root CA** sits at the **top of the trust hierarchy** — it is the anchor of the entire Public Key Infrastructure (PKI).
  * A **root certificate is always self-signed** because there is no higher authority to sign it. That’s why you might sometimes see warnings like “this certificate is self-signed.”
  * However, **self-signed is acceptable** when the certificate belongs to a **trusted root**, which is explicitly trusted by operating systems, browsers, and other clients.

  **Why Have a Separate Root and Intermediate CA?**
  * **Security:** The root CA’s private key is kept **offline or in secure environments**. Intermediate CAs handle day-to-day certificate issuance, minimizing exposure.
  * **Resilience:** If an **intermediate CA is compromised**, it can be revoked without affecting the root. This limits the blast radius.
  * **Best Practice:** This layered structure follows industry standards for security and trust management.

---

  **TLS Certificate Chain Flow (Using pinkbank.com as Example)**

  **Scenario:**

  Seema visits `pinkbank.com`, which uses a TLS certificate from Let’s Encrypt.

  **What Happens:**

  1. **pinkbank.com** presents:

      * Its **own certificate** (called the **leaf certificate**)
        e.g., `CN=pinkbank.com`
      * Its intermediate certificate, issued by Let’s Encrypt Root CA (e.g., ISRG Root X2) to Let’s Encrypt Intermediate CA (e.g., CN=R3 or CN=E1, depending on the chain used).

  2. The browser then:

      * **Verifies the signature on the leaf cert** using the **public key of the intermediate CA**.
      * **Verifies the intermediate CA’s certificate** using the **public key of the root CA**, which is already **preinstalled and trusted** in the browser’s trust store.

  3. The **Root CA** (e.g., `ISRG Root X2`) is:

      * **Not sent** by the server.
      * **Already trusted** by Seema's browser (this is why root CAs are preloaded in browser/OS trust stores).

  **The flow is:**

  ```
  pinkbank.com (leaf cert) 
    ⬅ signed by 
  Let's Encrypt Intermediate CA (R3/E1)
    ⬅ signed by 
  ISRG Root X2 (Root CA, already in browser)
  ```

  > When Seema's browser sees this chain, it uses the **root CA it already trusts** to verify the chain of trust. That’s why even though `pinkbank.com` doesn’t send the root cert, the validation still succeeds.

---


## CLIENT-SIDE: How the Scheduler Authenticates to the API Server

In mutual TLS, the **client also needs to present its identity** — in this case, the scheduler must prove who it is to the API server.

---

### Step 1: How Does the Scheduler Authenticate Itself?

Open the same kubeconfig file used by the scheduler:

```bash
cat /etc/kubernetes/scheduler.conf
```

Look for:

```yaml
users:
- name: system:kube-scheduler
  user:
    client-certificate-data: <base64 encoded cert>
    client-key-data: <base64 encoded key>
```

These fields hold:

* `client-certificate-data` → the scheduler’s **public certificate**
* `client-key-data` → the corresponding **private key**

To decode and inspect the certificate:

```bash
echo -n "<client-certificate-data>" | base64 --decode > scheduler.crt
openssl x509 -in scheduler.crt -text -noout
```

You should see something like:

```
Subject: CN=system:kube-scheduler
Issuer: CN=kubernetes
```

This confirms:

* The scheduler is identifying as `system:kube-scheduler`
* The certificate is signed by the **cluster CA**

---

### Step 2: How Does the API Server Verify the Scheduler?

To authenticate the client, the **API server must trust the CA** that signed the scheduler's certificate.

Check the API server manifest:

```bash
cat /etc/kubernetes/manifests/kube-apiserver.yaml
```

Look for:

```yaml
--client-ca-file=/etc/kubernetes/pki/ca.crt
```

This tells us:

* The API server uses `/etc/kubernetes/pki/ca.crt` to **validate client certificates**
* Since the scheduler’s certificate is signed by this CA, the API server **accepts and trusts** it

---

## Conclusion: Focus on Config Files, Not Hardcoded Paths

Here’s what you should take away:

* You **don’t need to memorize** certificate file paths
* Instead, focus on the **config files**:

  * Static pod manifests (`/etc/kubernetes/manifests/`) for server-side config
  * Kubeconfig files (e.g. `scheduler.conf`) for client-side authentication

Everything flows from there.

---

### Summary Table

| Component  | What It Does                          | Where to Look                                   | What to Look For                                                           |
| ---------- | ------------------------------------- | ----------------------------------------------- | -------------------------------------------------------------------------- |
| API Server | Presents TLS cert to clients          | `/etc/kubernetes/manifests/kube-apiserver.yaml` | `--tls-cert-file`, `--client-ca-file`                                      |
| Scheduler  | Verifies server, presents client cert | `/etc/kubernetes/scheduler.conf`                | `certificate-authority-data`, `client-certificate-data`, `client-key-data` |

---

## Example 2: mTLS Between API Server (Client) and etcd (Server)

![Alt text](/images/33c.png)

In this example, we’ll examine how **mutual TLS (mTLS)** works between the **kube-apiserver** (client) and **etcd** (server). We'll again use the principle of inspecting **configuration files** rather than memorizing certificate paths.

This reinforces the idea that once you know **where to look**, you can figure out **how any TLS-secured communication** works inside Kubernetes.

---

## SERVER-SIDE: etcd Presents Its Certificate to API Server

### Step 1: Know Who’s the Server

In this scenario:

* **etcd** is the **server**
* **kube-apiserver** is the **client**

The server (etcd) must present a valid TLS certificate to the API server, and the API server must verify it.

---

### Step 2: Where Is etcd's Certificate Defined?

We know from [Day 15](https://github.com/CloudWithVarJosh/CKA-Certification-Course-2025/tree/main/Day%2015) that etcd runs as a **static pod**. So, inspect the etcd manifest:

```bash
cat /etc/kubernetes/manifests/etcd.yaml
```

Look for the following flags:

```yaml
--cert-file=/etc/kubernetes/pki/etcd/server.crt
--key-file=/etc/kubernetes/pki/etcd/server.key
--client-cert-auth=true
--trusted-ca-file=/etc/kubernetes/pki/etcd/ca.crt
```

This tells us:

* etcd uses `/etc/kubernetes/pki/etcd/server.crt` as its TLS certificate.
* It trusts only client certificates signed by `/etc/kubernetes/pki/etcd/ca.crt`.

Inspect the certificate:

```bash
openssl x509 -in /etc/kubernetes/pki/etcd/server.crt -text -noout
```

You should see:

* **Subject:** `CN=my-second-cluster-control-plane` (matching the control plane hostname)
* **Issuer:** `CN=etcd-ca` (or possibly `CN=kubernetes`, depending on your setup)

This confirms:

* etcd is identifying itself as `my-second-cluster-control-plane`
* It’s using a certificate signed by a trusted Certificate Authority (CA)

---

### Step 3: How Does the API Server Verify etcd's Certificate?

Since the **API server is the client**, it needs to verify the certificate presented by etcd. To do this, it must trust the **CA that signed etcd’s server certificate**.

Check the API server’s static pod manifest:

```bash
cat /etc/kubernetes/manifests/kube-apiserver.yaml
```

Look for these flags:

```yaml
--etcd-cafile=/etc/kubernetes/pki/etcd/ca.crt
--etcd-certfile=/etc/kubernetes/pki/apiserver-etcd-client.crt
--etcd-keyfile=/etc/kubernetes/pki/apiserver-etcd-client.key
```

The `--etcd-cafile` flag tells us:

* The API server uses `/etc/kubernetes/pki/etcd/ca.crt` to verify etcd’s server certificate.
* This CA file must match the **issuer** of the certificate that etcd presents (i.e., `CN=etcd-ca`).

Inspect it with:

```bash
openssl x509 -in /etc/kubernetes/pki/etcd/ca.crt -text -noout
```

You should see:

* **Subject:** `CN=etcd-ca`
* **Issuer:** `CN=etcd-ca` (self-signed)

This confirms:

* The API server **trusts the etcd-ca**, and that’s how it verifies the authenticity of etcd’s certificate during the TLS handshake.

---


## CLIENT-SIDE: API Server Authenticates to etcd

### Step 1: Where Does API Server Specify TLS Credentials?

Let’s inspect how the API server connects securely to etcd.

Check the static pod manifest:

```bash
cat /etc/kubernetes/manifests/kube-apiserver.yaml
```

Look for the etcd-related TLS flags:

```yaml
--etcd-cafile=/etc/kubernetes/pki/etcd/ca.crt
--etcd-certfile=/etc/kubernetes/pki/apiserver-etcd-client.crt
--etcd-keyfile=/etc/kubernetes/pki/apiserver-etcd-client.key
```

Let’s break this down:

* `--etcd-cafile`: The **CA certificate** used by the API server to verify etcd’s identity.
* `--etcd-certfile`: The **client certificate** the API server presents to etcd.
* `--etcd-keyfile`: The **private key** associated with the above certificate.

Inspect the client certificate:

```bash
openssl x509 -in /etc/kubernetes/pki/apiserver-etcd-client.crt -text -noout
```

You should see:

* **Subject:** `CN=kube-apiserver-etcd-client`
* **Issuer:** `CN=etcd-ca`

This confirms:

* The API server authenticates to etcd using the identity `kube-apiserver-etcd-client`
* The certificate is signed by the **etcd-ca**, which etcd is configured to trust

---

### Step 2: How Does etcd Verify the API Server's Certificate?

Since **etcd is the server**, it must validate the client certificate presented by the API server.

Look at etcd’s static pod manifest:

```bash
cat /etc/kubernetes/manifests/etcd.yaml
```

Look for this flag:

```yaml
--trusted-ca-file=/etc/kubernetes/pki/etcd/ca.crt
```

This tells us:

* etcd will **only accept client certificates** that are signed by the **CA in `etcd/ca.crt`** — which is the same CA that issued the API server’s `kube-apiserver-etcd-client` certificate.

So:

* etcd uses `--trusted-ca-file` to verify the API server’s certificate.
* The certificate is trusted because it’s signed by the `etcd-ca`.

---

### How Mutual TLS (mTLS) Works Here

* **etcd (server)** presents `server.crt`, and **API server (client)** verifies it using `--etcd-cafile`
* **API server (client)** presents `apiserver-etcd-client.crt`, and **etcd (server)** verifies it using `--trusted-ca-file`

This is **mutual TLS** — both components **authenticate each other** using certificates.

---

### Takeaway: Focus on Flags, Not File Paths

You don’t need to memorize paths. What matters is:

* Who is the **client**, who is the **server**
* Which component is **presenting** a certificate
* Which component is **validating** that certificate via a **trusted CA**

The static pod manifests show you everything you need.

---

### Summary Table

| Component      | Role   | Where to Look                                   | What to Look For                                     |
| -------------- | ------ | ----------------------------------------------- | ---------------------------------------------------- |
| etcd           | Server | `/etc/kubernetes/manifests/etcd.yaml`           | `--cert-file`, `--key-file`, `--trusted-ca-file`     |
| kube-apiserver | Client | `/etc/kubernetes/manifests/kube-apiserver.yaml` | `--etcd-cafile`, `--etcd-certfile`, `--etcd-keyfile` |

---


## Example 3: mTLS Between API Server (Client) and Kubelet (Server)

![Alt text](/images/33c.png)

In this example, we will explore **how mutual TLS (mTLS)** works between the **API server** (client) and the **kubelet** (server).

This is a vital communication channel, especially for actions like executing commands in a pod (`kubectl exec`), fetching logs (`kubectl logs`), and metrics collection — all of which require the API server to securely interact with the kubelet running on each node.

Like before, we'll **discover certificate paths and roles by reading configuration files**, rather than relying on memorization.

---

## Who is the Client and Who is the Server?

* **API Server** → acts as the **client**
* **Kubelet** → acts as the **server**

---

## SERVER-SIDE: How the Kubelet Presents Its Certificate

Before diving in, it's important to understand:

> Unlike the API server, controller manager, etcd, or scheduler — **the kubelet does not run as a Pod**.
> It runs as a **systemd-managed process** on the node and is typically located at `/usr/bin/kubelet`.
> That means there’s no Pod manifest to inspect — instead, we must look at the **process arguments** to understand where kubelet stores its certificates and config.

---

### Step 1: Locate the Kubelet Process

Start by checking how kubelet is launched:

```bash
ps -ef | grep kubelet
```

In the output, you’ll typically find a flag like:

```bash
--config=/var/lib/kubelet/config.yaml
```

This tells us that kubelet is using the config file at:

```bash
/var/lib/kubelet/config.yaml
```

From this, we learn an important principle:

> The presence of the config file at `/var/lib/kubelet/config.yaml` strongly suggests that **kubelet-related files (certs, kubeconfigs, etc.) are stored under `/var/lib/kubelet/`**.

Even if the `config.yaml` doesn’t explicitly list `certDirectory`, you can navigate to:

```bash
/var/lib/kubelet/pki/
```

Inside this directory, you’ll typically find:

| File                             | Purpose                                             |
| -------------------------------- | --------------------------------------------------- |
| `kubelet.crt`                    | Server certificate (when kubelet acts as a server)  |
| `kubelet.key`                    | Private key for `kubelet.crt`                       |
| `kubelet-client-current.pem`     | Client cert (used when kubelet talks to API server) |
| `kubelet-client-<timestamp>.pem` | Rotated historical client certs                     |

For **server-side TLS**, the file of interest is:

```bash
/var/lib/kubelet/pki/kubelet.crt
```

This is the certificate the **kubelet presents when the API server connects to it** — such as during `kubectl exec`, `logs`, etc.

---

### Step 2: Inspect the Server Certificate

We're interested in how kubelet identifies itself as a **server**, so inspect:

```bash
openssl x509 -in /var/lib/kubelet/pki/kubelet.crt -text -noout
```

You may find something like:

```
Subject: CN=system:node:<node-name>, O=system:nodes
Issuer: CN=my-second-cluster-control-plane-ca@1741433227
```

This confirms:

* The **Common Name (CN)** is `system:node:<node-name>`, showing it's a kubelet identity.
* It’s signed by a **cluster CA** named `my-second-cluster-control-plane-ca@1741433227`.

This is the **CA Kubernetes used to sign the kubelet’s server certificate** — ensuring trust.

---

###  How Did the Kubelet Get This Certificate?

Initially, the kubelet doesn’t have a certificate. Instead, it uses a **bootstrap token** to authenticate and request a signed certificate from the API server.

This mechanism is enabled in the API server by this flag:

```yaml
--enable-bootstrap-token-auth=true
```

So, kubelet starts by using a bootstrap token, gets authenticated by the API server, and then receives a **signed TLS certificate** — which becomes the `kubelet.crt`.

---

### Step 3: How the API Server Verifies the Kubelet (or Doesn’t)

In this connection, the **API server acts as the client**, initiating HTTPS requests to the kubelet. Normally, TLS clients verify the server’s certificate — but here’s what happens in Kubernetes:

Inspect the kubelet’s server certificate again:

```bash
openssl x509 -in /var/lib/kubelet/pki/kubelet.crt -text -noout
```

You’ll see:

```
Issuer: CN=my-second-cluster-control-plane-ca@1741433227
```

So the kubelet’s certificate is signed by a valid internal CA. But here’s the catch:

> The API server is **not configured to trust this CA**.

There is **no `--kubelet-certificate-authority` flag** in the API server’s manifest. That means the API server does **not validate the kubelet’s certificate** in the traditional TLS sense.

Yet the connection still works. Why?

**Because the trust here is enforced at the client-auth side**:

* The **API server presents a client certificate**, configured via:

  ```yaml
  --kubelet-client-certificate
  --kubelet-client-key
  ```

* The **kubelet verifies this certificate** using its `--client-ca-file`

So while the **kubelet validates the API server**, the reverse is **not strictly true**.

> This isn’t full mutual TLS — and this asymmetry is a known behavior in kubeadm-based clusters.

The API server **relies on RBAC, Node authorizer, and NodeRestriction** for secure interactions with the kubelet, not on validating its TLS server certificate.

---

## CLIENT-SIDE: How the API Server Presents Its Own Certificate to the Kubelet

This is **mutual TLS**, so the API server must also **authenticate itself** to the kubelet.

In the `kube-apiserver` manifest, you’ll find:

```yaml
--kubelet-client-certificate=/etc/kubernetes/pki/apiserver-kubelet-client.crt
--kubelet-client-key=/etc/kubernetes/pki/apiserver-kubelet-client.key
```

Inspect the client certificate:

```bash
openssl x509 -in /etc/kubernetes/pki/apiserver-kubelet-client.crt -text -noout
```

You’ll likely see something like:

```
Subject: CN=kube-apiserver-kubelet-client
Issuer: CN=kubernetes
```

This means:

* The API server presents itself as `kube-apiserver-kubelet-client`
* The certificate is issued by the **Kubernetes cluster CA**

---

### How the Kubelet Verifies the API Server's Certificate

To enforce authentication, the **kubelet must verify that the client (API server) is trusted**.

Open the kubelet config file:

```bash
cat /var/lib/kubelet/config.yaml
```

You’ll find:

```yaml
clientCAFile: /etc/kubernetes/pki/ca.crt
```

This tells us:

* The kubelet uses `/etc/kubernetes/pki/ca.crt` as its **client certificate authority**
* Any client (like the API server) presenting a certificate must be **signed by this CA**

So when the API server connects and presents its certificate:

* Subject: `CN=kube-apiserver-kubelet-client`
* Issuer: The CA in `/etc/kubernetes/pki/ca.crt`

The kubelet validates the certificate using this CA file and accepts the connection.

> This is how **certificate-based client authentication** is enforced on the kubelet side.

---

## Summary: mTLS Between API Server and Kubelet

| Role       | Component  | Certificate File                                   | Notes                                                     |
| ---------- | ---------- | -------------------------------------------------- | --------------------------------------------------------- |
| Server     | Kubelet    | `/var/lib/kubelet/pki/kubelet.crt`                 | Signed by `my-second-cluster-control-plane-ca@1741433227` |
| Client     | API Server | `/etc/kubernetes/pki/apiserver-kubelet-client.crt` | Presented to kubelet to authenticate                      |
| Trust Root | Kubelet    | `/etc/kubernetes/pki/ca.crt`                       | Used to verify API server’s client certificate            |
| (Optional) | API Server | *Not enforced*                                     | Does not strictly verify the kubelet’s certificate        |

---

##  Recap: How mTLS Happens Here

1. The **kubelet** starts with a **bootstrap token**, then obtains a **server certificate** signed by the cluster CA.
2. The **API server connects** to the kubelet, and although no strict verification is enforced, the kubelet's certificate may be validated **implicitly** if the TLS library uses `/etc/kubernetes/pki/ca.crt`.
3. The **API server presents its client certificate** (`apiserver-kubelet-client.crt`) to the kubelet.
4. The **kubelet verifies** this certificate against its configured CA (`clientCAFile: /etc/kubernetes/pki/ca.crt`).
5. **mTLS is established** — although only the kubelet strictly verifies the peer’s certificate.

---

##  Tasks for You

Now that you’ve seen how mTLS works between core Kubernetes components, here are **3 hands-on tasks** to apply what you’ve learned:

#### Task 1: Controller Manager (Client) → API Server (Server)

This is similar to Example 1 (Scheduler → API Server).
Your goal:

* Identify the certificate the **Controller Manager** uses as a client.
* Verify the **API Server’s certificate and CA** used on the server side.
* Confirm how trust is established.

Hint: Start by checking the `kube-controller-manager` manifest file.

---

####  Task 2: Kubelet (Client) → API Server (Server)

You already saw kubelet acting as a server — now reverse the flow.

* How does the **kubelet authenticate to the API Server**?
* Where is the client cert stored?
* How does the API server validate it?

Look for flags like `--kubeconfig` in the kubelet's launch config.

---

####  Task 3: Kube-Proxy (Client) → API Server (Server)

This one’s a bit more involved — kube-proxy runs as a **DaemonSet**.

* SSH into a node or use `kubectl exec`  to explore a **kube-proxy pod**.
* Find its **kubeconfig file**.
* Inspect the client certificate it uses.
* Confirm how the **API server authenticates** this client.

 **Challenge**: For this task, you'll need to **research on your own**. Use official Kubernetes docs, GitHub, or community threads. This is an essential skill when dealing with real-world clusters.

---

### **Granting Cluster Access to a New User (Seema) using Certificates and RBAC**

![Alt text](/images/34a.png)

---

**Granting Cluster Access to a New User (Seema) using Certificates and RBAC**
To securely grant a new user like **Seema** access to a Kubernetes cluster, we follow a series of steps involving certificate-based authentication and Role-Based Access Control (RBAC). This ensures Seema can connect and interact with the cluster within a defined scope.

---

**Step 1: Seema Generates a Private Key**

```bash
openssl genrsa -out seema.key 2048
```

This generates a 2048-bit RSA **private key**, saved to `seema.key`. This key will be used to generate a certificate signing request (CSR) and later to authenticate to the Kubernetes cluster. It must remain **private and secure**.

---

**Step 2: Seema Generates a Certificate Signing Request (CSR)**

```bash
openssl req -new -key seema.key -out seema.csr -subj "/CN=seema"
```

Seema uses her private key to create a **CSR**. The `-subj "/CN=seema"` sets the **Common Name (CN)** to `seema`, which becomes her Kubernetes username. The generated CSR contains her public key and identity, and will be signed by a Kubernetes cluster admin.

---

**Step 3: Seema Shares the CSR with the Kubernetes Admin**

```bash
cat seema.csr | base64 | tr -d "\n"
```

The CSR must be **base64-encoded** to embed it into a Kubernetes object. This command converts the CSR into a single-line base64 string, stripping newlines with `tr -d "\n"`—a necessary step for YAML formatting.

---

**Step 4: Kubernetes Admin Creates the CSR Object in Kubernetes**

```yaml
apiVersion: certificates.k8s.io/v1
kind: CertificateSigningRequest
metadata:
  name: seema
spec:
  request: <BASE64_ENCODED_CSR>
  signerName: kubernetes.io/kube-apiserver-client
  expirationSeconds: 7776000
  usages:
  - client auth
```

The admin creates a Kubernetes `CertificateSigningRequest` object.

* `request` is the base64-encoded CSR.
* `signerName: kubernetes.io/kube-apiserver-client` instructs Kubernetes to treat this as a **client authentication** request.
* `usages` defines that this certificate will be used for **client authentication**, not server TLS or other use cases.
* `expirationSeconds` sets the certificate’s validity to **90 days** (7776000 seconds).

---

**Step 5: Kubernetes Admin Approves the CSR**

```bash
kubectl certificate approve seema
```

This command **approves and signs** the certificate request. Kubernetes issues a certificate for Seema, valid per the defined usage and expiration settings.

---

**Step 6: Admin Retrieves and Shares the Signed Certificate**

```bash
kubectl get csr seema -o jsonpath='{.status.certificate}' | base64 -d > seema.crt
```

The admin retrieves the **signed certificate** from the CSR’s status, decodes it from base64, and saves it as `seema.crt`. This certificate, along with `seema.key`, is sent back to Seema for kubeconfig configuration.

---

**Step 7: Seema Configures Her `kubeconfig` with Credentials and Cluster Info**

```bash
kubectl config set-credentials seema \
  --client-certificate=seema.crt \
  --client-key=seema.key \
  --certificate-authority=ca.crt \
  --embed-certs=true

kubectl config set-cluster kind-my-second-cluster \
  --server=https://127.0.0.1:59599 \
  --certificate-authority=ca.crt \
  --embed-certs=true \
  --kubeconfig=~/.kube/config

kubectl config set-context seema@kind-my-second-cluster-context \
  --cluster=kind-my-second-cluster \
  --user=seema \
  --namespace=default
```

These commands configure the **user credentials**, **cluster endpoint**, and **context** in Seema’s `kubeconfig`:

* The first command tells kubectl how to authenticate Seema using her certificate/key.
* The second registers the cluster endpoint using the correct CA.
* The third defines a context associating the user, cluster, and default namespace.

---

**Step 8: Admin Creates a Role and RoleBinding for Seema**

```yaml
kind: Role
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  namespace: default
  name: seema-role
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "list", "delete"]
```

```bash
kubectl create rolebinding seema-binding \
  --role=seema-role \
  --user=seema \
  --namespace=default
```

The **Role** allows Seema to **get**, **list**, and **delete** pods in the `default` namespace.
The **RoleBinding** assigns this Role to Seema's username (`CN=seema`), authorizing her actions.

---

**Step 9: Admin Verifies Authorization with `can-i`**

```bash
kubectl auth can-i delete pods --namespace=default --as=seema
```

This command is run by the **admin** to simulate whether Seema is allowed to **delete pods** in the `default` namespace.

* `--as=seema` impersonates Seema’s user identity.
* This confirms that the **RBAC permissions** are set correctly before Seema starts using the cluster.

---

**Step 10: Seema Switches to Her Configured Context**

```bash
kubectl config use-context seema@kind-my-second-cluster-context
```

This sets Seema's **active context** to the one defined earlier, allowing `kubectl` to use her certificate and connect to the right cluster/namespace.

---

**Optional: Use REST API or Alternate `kubeconfig` Files**

```bash
curl https://<API-SERVER-IP>:<PORT>/api/v1/namespaces/default/pods \
  --cacert ca.crt --cert seema.crt --key seema.key
```

*Seema can authenticate with the cluster directly via API using her certificate.*

```bash
kubectl get pods --kubeconfig=myconfig.yaml
```

*She can manage multiple clusters by specifying alternate kubeconfig files.*

---

**Step 11: Check Certificate Expiry**

```bash
openssl x509 -noout -dates -in seema.crt
```

This displays the `notBefore` and `notAfter` dates for the certificate, helping Seema monitor its expiration.

---

## Conclusion

By now, you’ve gained a solid, practical understanding of how **TLS encryption** works across Kubernetes—from the basics of symmetric and asymmetric encryption, to the internals of **mutual TLS (mTLS)** between control plane components, and even how **users authenticate using certificates and RBAC**.

This wasn’t about memorizing flags or file paths. It was about building an intuition for:

* Who acts as a **client** or **server** in Kubernetes communications
* How **kubeconfig** files drive TLS-based authentication
* Why **private CAs** are central to cluster trust
* How to **inspect and verify** certificate flows using real config files

Most importantly, you’ve now seen how these concepts map to actual commands, manifests, and security practices used in real Kubernetes clusters—be it self-hosted or managed.

This foundational TLS knowledge is not only **exam-relevant (CKA, CKS)** but also essential for **day-to-day Kubernetes operations**, **troubleshooting**, and **cluster hardening**.

Whether you choose to revisit the videos in parts or prefer this full deep dive, you're now better equipped to explore Kubernetes security **with confidence and clarity**.

---

### References

* [Kubernetes: Authenticating](https://kubernetes.io/docs/reference/access-authn-authz/authentication/)
* [Kubernetes: Using RBAC Authorization](https://kubernetes.io/docs/reference/access-authn-authz/rbac/)
* [CertificateSigningRequest API](https://kubernetes.io/docs/reference/kubernetes-api/authentication-resources/certificate-signing-request-v1/)
* [Configure Access to Multiple Clusters](https://kubernetes.io/docs/concepts/configuration/organize-cluster-access-kubeconfig/)
* [kubectl config](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#config)
* [Kubernetes TLS Overview](https://kubernetes.io/docs/concepts/cluster-administration/certificates/)
* [Kubeconfig Explained](https://kubernetes.io/docs/concepts/configuration/organize-cluster-access-kubeconfig/)
* [TLS Bootstrapping](https://kubernetes.io/docs/reference/command-line-tools-reference/kubelet-tls-bootstrapping/)
* [Kubernetes: Authentication](https://kubernetes.io/docs/reference/access-authn-authz/authentication/)
* [Kubernetes: Using RBAC Authorization](https://kubernetes.io/docs/reference/access-authn-authz/rbac/)
* [CertificateSigningRequest API](https://kubernetes.io/docs/reference/kubernetes-api/authentication-resources/certificate-signing-request-v1/)

---
