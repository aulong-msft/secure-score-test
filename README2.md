# Security Plan – [sp-project-name]

## Preamble

CSE cannot certify/attest to security of an architecture nor code – the below is intended to help identify and track design decisions and outstanding work related to discussed attack vectors identified during the engagement and intended to provide guidance to CSE dev crews.

This template is intended to help assess security risk within customer engagement and produces backlog recommendations to mitigate that risk. Security reommendations can be used in conjunction with those suggested via [BRIE (https://aka.ms/BRIE)](https://aka.ms/BRIE).

## Overview

Please find the Threat Model for [sp-project-name] below. This document shows the threat model and data flow diagram of the application. These artifacts were constructed based on documentation and source code from the project itself and are subject to change as the architecture and codebase evolves. Each of the labeled entities in the figures below are accompanied by meta-information which describes the threats, describes the data in scope, and recommendations for security controls.

## Diagrams

### Architecture Diagram

![Architecture Diagram](./architecture-diagram.png)

### Data Flow Diagram

![Data flow diagram](./data-flow-diagram.png)

#### Data Flow Attributes

| # | Transport Protocol | Data Classification | Authentication | Notes |
| :---: | :--- | :--- | :--- | :--- |
| 1  | HTTPS/TLS 1.2 | Confidential  | API Key | List of operations are sent to the device. No personal data is attached to the payload. |
| 2 | ... | ... | ...  | ... |

### Threat Map

![Threat map](./threat-map.png)

### Threat Properties

---
**Threat #:** 1  
**Principle:** Confidentiality and Integrity  
**Affected Asset:** Inbound Internet connections  
**Threat:** As a result of the vulnerability of not encrypting data, plaintext data could be intercepted during transit via a man-in-the-middle (MitM) attack. Sensitive data could be exposed or tampered to allow further exploits.

**Mitigation:**

All products and services must encrypt data in transit using approved cryptographic protocols and algorithms.

1. Use TLS to encrypt all HTTP-based network traffic. Use other mechanisms, such as IPSec, to encrypt non-HTTP network traffic that contains customer or confidential data.
2. Use only TLS 1.2 or TLS 1.3. Use ECDHE-based ciphers suites and NIST curves. Use strong keys. Enable HTTP Strict Transport Security (HSTS). Turn off TLS compression and do not use ticket-based session resumption.

**Project Specific Guidance:**

For HTTPS message encryption, enforce a minimum version of TLS 1.2. This applies to all traffic illustrated in the diagram.

---
**Threat #:** 2  
**Principle:** Confidentiality  
**Affected Asset:** All data stores  
**Threat:** Data is a valuable target for most threat actors and attacking the data store directly, as opposed to stealing it during transit, allows data exfiltration at a much larger scale.

**Mitigation:**

All customer or confidential data must be encrypted before being written to non-volatile storage media (encrypted at-rest) per the following requirements.

1. Use approved algorithms. This includes AES-256, AES-192, or AES-128.
2. Leverage SQL TDE whenever available.
3. Encryption must be enabled before writing data to storage.

- Applies to all data stores on the diagram.
- Azure SQL Database encrypt data at rest by default using Transparent Data Encryption.

**Project Specific Guidance:**

- The SQL server and Blob Storage used in this model encrypt data at rest by default.
- The SQL Server instances should have TDE enabled.
- If utilizing IaC products, ensure the configuration files are stored in a storage account or other safe location and not in your repository.

---
**Threat #:** 3  
**Principle:** Confidentiality  
**Affected Asset:** All services  
**Threat:** Broken or non-existent authentication mechanisms may allow attackers to gain access to confidential information.

**Mitigation:**

All services within the Azure Trust Boundary must authenticate all incoming requests, including requests coming from the same network. Proper authorizations should also be applied to prevent unnecessary privileges.

- Whenever available, use Azure Managed Identities to authenticate services. Service Principals may be used if Managed Identities are not supported.
- External users or services may use Username + Passwords, Tokens, or Certificates to authenticate, provided these are stored on Key Vault or any other vaulting solution.
- For authorization, use Azure RBAC to segregate duties and grant only the least amount of access to perform an action at a particular scope.

**Project Specific Guidance:**

- Use either MFA or Service Principals and tokens to authenticate all Azure services in this project.
- Use Azure RBAC to grant the least number of privileges to perform an action at a particular scope.
- Enable MFA.
- Leverage AAD PIM for any administrative access.
- Capture Activity Logs to Log Analytics.
- Leverage managed identity wherever able.

---
**Threat #:** 4  
**Principle:** Confidentiality and Integrity  
**Affected Asset:** All services  
**Threat:** A large attack surface, particularly those that are exposed on the internet, will increase the probability of a compromise.

**Mitigation:**

Minimize the application attack surface by limiting publicly exposed services.

- Use strong network controls by using Azure Virtual Networks, Network Security Groups (NSG), or Private Endpoints to protect against unsolicited traffic.
- Use Azure Private Endpoints to block all internet connections to services that do not need to be publicly exposed.

**Project Specific Guidance:**

- Use Private Endpoints to protect Azure Blob Storage and Azure SQL. A private endpoint will wrap each service in its own VNET to securely interface with the network and connect each service privately and securely.  Additional Network Security Groups and controls can be implemented to ensure only desired traffic can enter and leave the system.
- Turning on Azure Defender will provide a list of vulnerabilities associated with your subscription, it is advised to turn on this feature (free 30-day trial) to obtain a list of security measures and mitigate through that list.

---
**Threat #:** 5  
**Principle:** Integrity  
**Affected Asset:** All services  
**Threat:** Exploitation of insufficient logging and monitoring is the bedrock of nearly every major incident.
Attackers rely on the lack of monitoring and timely response to achieve their goals without being detected.

**Mitigation:**

Logging of critical application events must be performed to ensure that, should a security incident occur, incident response and root-cause analysis may be done. Steps must also be taken to ensure that logs are available and cannot be overwritten or destroyed through malicious or accidental occurrences.

At a minimum, the following events should be logged.

1. Login/logout events
2. Password change events (if not integrated into SSO)
3. Privilege delegation events
4. Security validation failures (e.g., input validation or authorization check failures)
5. Application errors and system events
6. Application and system startups and shutdowns, as well as logging initialization

**Project Specific Guidance:**

- Implement a monitoring solution such as Azure Monitor or Log Analytics to monitor the web applications, security events, analytics, and workloads, respectively.

---
**Threat #:** 6  
**Principle:** Confidentiality and Integrity  
**Affected Asset:** All services  
**Threat:** Secrets leaking into unsecured locations are an easy way for adversaries to gain access to a system. These secrets can be used to either spoof the owners of these secrets or, in the case of encryption keys, use them to decrypt data.

**Mitigation:**

Proper storage and management of secrets is critical in protecting systems from compromises, in most cases, with severe impact.

1. Never store secrets in code or configuration files. Instead, use a vault or any secure container (such as encrypted variables) to store secrets.
2. Separate application secrets by environment.
3. Rotate all secrets before turning over the application to the customer.

**Project Specific Guidance:**

- Store all secrets, encryption keys and certificates in Key Vault.

---
**Threat #:** 7  
**Principle:** Confidentiality and Integrity  
**Affected Asset:** CI/CD Pipelines  
**Threat:** CI/CD Pipelines should always be cleansed of sensitive information being leaked.

**Mitigation:**

All secrets, key materials, and credentials stored in CI/CD Pipelines should always be cleansed of sensitive information that can be witnessed in plaintext.

**Project Specific Guidance:**

- Add static code analysis tools in CI/CD pipelines to gain security insights about the code being developed, gate insecure code from entering production, and generate an overall more secure and complete solution.
- Add dependency scanning to CI/CD pipelines to ensure you are using the most current and secure libraries.
- Add credential scanning to CI/CD pipelines to ensure secrets, key information, and passwords are not leaked into the open.

---
**Threat #:** 8  
**Principle:** Availability  
**Affected Asset:** All services  
**Threat:** Distributed denial of service (DDoS) attacks is some of the largest availability and security concerns facing customers that are moving their applications to the cloud. A DDoS attempts to exhaust an application's resources, making the application unavailable to legitimate users.

**Mitigation:**

DDoS protection is important to ensure the availability of your cloud services. Azure provides DDoS solutions to you and your customers to provide extra safeguards to this popular attack.

**Project Specific Guidance:**

- turn on DDoS protection feature.
- DDoS Basic handles large DDoS attacks restrict to VNet for further isolation.

---
**Threat #:** 9  
**Principle:** Privacy  
**Affected Asset:** All data stores  
**Threat:** The application does not allow the deletion of a user’s profile from storage.

**Mitigation:**

Implement the ability to delete a user’s profile, and all associated data, from the datastores.

---

## Appendix

### Security Principles

---

#### **Confidentiality**

Confidentiality refers to the objective of keeping data private or secret. In practice, it’s about controlling access to data to prevent unauthorized disclosure.

#### **Integrity**

Integrity is about ensuring that data has not been tampered with and, therefore, can be trusted. It is correct, authentic, and reliable.

#### **Avaliability**

 Availability means that networks, systems, and applications are up and running. It ensures that authorized users have timely, reliable access to resources when they are needed.

### Microsoft Zero Trust Principles
  
---

#### **Verify explicitly**
  
Always authenticate and authorize based on all available data points, including user identity, location, device health, service or workload, data classification, and anomalies.

#### **Use least privileged access**
  
Limit user access with just-in-time and just-enough-access (JIT/JEA), risk-based adaptive policies, and data protection to help secure both data and productivity.

#### **Assume breach**
  
Minimize blast radius and segment access. Verify end-to-end encryption and use analytics to get visibility, drive threat detection, and improve defenses.

### Commercial Data Classification Reference

---

#### **Sensitive**  

Data that is to have the most limited access and requires a high degree of integrity. This is typically data that will do the most damage to the organization should it be disclosed.  
Personal data (including PII) falls into this category and includes any identifier, such as name, an identification number, location data, online identifier. This also includes data related to one or more factors specific to the physical, psychological, genetic, mental, economic, cultural, or social identity of an individual.

#### **Confidential**

Data that might be less restrictive within the company but might cause damage if disclosed.

#### **Private**

Private data is usually compartmental data that might not do the company damage but must be keep private for other reasons. Human resources data is one example of data that can be classified as private.

#### **Proprietary**

Proprietary data is data that is disclosed outside the company on a limited basis or contains information that could reduce the company's competitive advantage, such as the technical specifications of a new product.

#### **Public**

Public data is the least sensitive data used by the company and would cause the least harm if disclosed. This could be anything from data used for marketing to the number of employees in the company.
