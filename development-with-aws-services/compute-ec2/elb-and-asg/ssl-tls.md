---
icon: file-certificate
---

# SSL/TLS

* **SSL** is deprecated and **not recommended** for use due to security vulnerabilities.
* **TLS** is the modern standard, more secure and widely used today
  * Uses **stronger encryption algorithms**, such as AES (Advanced Encryption Standard)
  * Addressed and patched several vulnerabilities that were present in **SSL**, including the **POODLE (**Padding Oracle On Downgraded Legacy Encryption**)**, **BEAST(**Browser Exploit Against SSL/TLS**)**, and **RC4 bias**

1. **Certificate Authority (CA)**: A trusted organization called a Certificate Authority (like Let’s Encrypt, DigiCert, or Comodo) issues certificates.
2. **Request Process**:
   * A website owner generates a **Certificate Signing Request (CSR)**, which includes the website’s public key and identifying information (domain name, company).
   * This CSR is sent to the CA, who verifies the requester’s identity and the domain ownership.
3. **CA Signs the Certificate**:
   * If the information checks out, the CA uses its own private key to sign the website’s certificate, creating a **digital signature** that certifies its authenticity.
4. **Expiration Date**: Certificates have a defined **validity period** (usually 1 to 2 years, though shorter terms are becoming more common).

## Implementing SSL/TLS on AWS -> decrypt tracffic at LB level <a href="#fd12" id="fd12"></a>



**1. Obtain an SSL/TLS Certificate**

* AWS Certificate Manager (ACM) to get a certificate. ACM provides a convenient and cost-effective way to manage SSL/TLS certificates within the AWS ecosystem.
* A third-party certificate authority (CA).&#x20;
* **X.509 Certificates**: An X.509 certificate is a **digital document** that verifies the identity of an entity (like a website or server) and provides the **public key** used for encrypted communication.
* **Used in SSL/TLS**: SSL/TLS relies on X.509 certificates to authenticate servers and, optionally, clients. When a load balancer uses SSL/TLS, it generally holds an X.509 certificate that authenticates it to clients.

**2. Attach the Certificate to Your Load Balancer**

Once you have your SSL/TLS certificate, the next step is to attach it to yourALB.&#x20;

* Navigate to the AWS Management Console.
* Open the EC2 Dashboard and select “Load Balancers.”
* Choose your load balancer and go to the “Listeners” tab.
* Edit the HTTPS listener and select “Change” to attach your SSL/TLS certificate.
* Clients can use SNI ( Server Name Indication) to specify hostname

<details>

<summary>SNI</summary>

**Server Name Indication (SNI)** is an extension of the **TLS/SSL protocol** that allows a client (like a web browser) to indicate which hostname it’s trying to connect to at the beginning of the **SSL/TLS handshake**. This is crucial when a single server (or load balancer) hosts multiple domains with separate SSL/TLS certificates.

* With SNI, each domain can have its own **individual certificate**, enhancing security by not forcing all domains to use a single certificate

With ALB you can configure multiple certificates for different domains, with one certificate designated as the **default**.

**NLBs do not support the concept of a default certificate** in the same way as ALBs. For NLBs, you would generally need to configure **specific SSL certificates for each domain**. If no match is found, the connection would fail.

#### How SNI Works:

* **Client Request**: When a client connects to a server over HTTPS, it includes the domain name in the initial handshake using SNI. _Newer protocol required client to indicate the hostname of the target server in the initial handshake_
* **Server Response**:  load balancer reads the hostname sent by the client and presents the correct SSL/TLS certificate for that specific domain or return ( fall back) the default. If&#x20;

</details>



**3. Configure SSL/TLS Termination**

SSL/TLS termination involves decrypting traffic at the load balancer level before forwarding it to your backend servers. This allows your backend servers to receive unencrypted traffic, reducing the processing load on them. **Termination** means that the **SSL/TLS encryption is handled (or "terminated") by the load balancer** instead of each backend server.

**4. Enforce HTTPS**

**5. Regularly Update and Monitor Your Certificates**

SSL/TLS certificates have expiration dates, so it’s essential to monitor their validity and renew them before they expire.&#x20;
