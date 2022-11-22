# Threat Model – [Project Name]

Please modify the template as necessary  to add additional information and remove information that is out of scope.

## Preamble

CSE cannot certify/attest to security of an architecture nor code – the below is intended to help identify and track design decisions and outstanding work related to discussed attack vectors identified during the engagement and intended to provide guidance to CSE dev crews.

This template is intended to help assess security risk within customer engagement and produces backlog recommendations to mitigate that risk. Security reommendations can be used in conjunction with those suggested via [BRIE (https://aka.ms/BRIE)](https://aka.ms/BRIE).


## Overview

Please find the Threat Model for [Project Name] below. This document shows the threat model and data flow diagram of the application. These artifacts were constructed based on documentation and source code from the project itself and are subject to change as the architecture and codebase evolves. Each of the labeled entities in the figures below are accompanied by meta-information which describes the threats, describes the data in scope, and recommendations for security controls.

## Diagrams

### Architecture Diagram

![Architecture Diagram]<add diagram image here>  

### Data Flow Diagram

![Data Flow Diagram]<add diagram image here>

### Data Flow Diagram Attributes
  
| # | Transport Protocol | Data Classification | Authentication | Notes|
|---|--------------------|---------------------|----------------|------|
| 1 | [Name of Protocol] | [Classification]    |  [AuthN]       | [Additonal Notes]


### Threat Properties
---
| # | Security Principle | Threat| Mitigation |
|---|--------------------|-------|------------|
| 1 |  Confidentiality and Integrity  | As a result of the vulnerability of not encrypting data, plaintext data could be intercepted during transit via a man-in-the-middle (MitM) attack. Sensitive data could be exposed or tampered to allow further exploits. | All products and services must encrypt data in transit using approved cryptographic protocols and algorithms. <ul><li>Use TLS to encrypt all HTTP-based network traffic. Use other mechanisms, such as IPSec, to encrypt non-HTTP network traffic that contains customer or confidential data. </li><li>Use only TLS 1.2 or TLS 1.3. Use ECDHE-based ciphers suites and NIST curves. Use strong keys. Enable HTTP Strict Transport Security (HSTS). Turn off TLS compression and do not use ticket-based session resumption.</li></ul> |
| 2 | Confidentiality and Integrity | As a result of the vulnerability of not encrypting data, plaintext data could be intercepted during transit via a man-in-the-middle (MitM) attack. Sensitive data could be exposed or tampered to allow further exploits. | All products and services must encrypt data in transit using approved cryptographic protocols and algorithms. <ul><li>Use TLS to encrypt all HTTP-based network traffic. Use other mechanisms, such as IPSec, to encrypt non-HTTP network traffic that contains customer or confidential data.</li><li> Use only TLS 1.2 or TLS 1.3. Use ECDHE-based ciphers suites and NIST curves. Use strong keys. Enable HTTP Strict Transport Security (HSTS). Turn off TLS compression and do not use ticket-based session resumption.</li></ul>
| 3 | Confidentiality | Data is a valuable target for most threat actors and attacking the data store directly, as opposed to stealing it during transit, allows data exfiltration at a much larger scale. | All customer or confidential data must be encrypted before being written to non-volatile storage media (encrypted at-rest) per the following requirements. <ul><li>Use approved FIPS compliant algorithms. This includes AES-256, AES-192, or AES-128.</li><li>Leverage SQL TDE whenever available.</li><li>Encryption must be enabled before writing data to storage.</li></ul>
| 4 | Confidentiality  | Broken or non-existent authentication mechanisms may allow attackers to gain access to confidential information. | All services within the Azure Trust Boundary must authenticate all incoming requests, including requests coming from the same network. Proper authorizations should also be applied to prevent unnecessary privileges. <ul><li>Whenever available, use Azure Managed Identities to authenticate services. Service Principals may be used if Managed Identities are not supported.</li><li>External users or services may use Username + Passwords, Tokens, or Certificates to authenticate, provided these are stored on Key Vault or any other vaulting solution.</li></li><li>For authorization, use Azure RBAC to segregate duties and grant only the least amount of access to perform an action at a particular scope.</li></ul>
| 5 | Confidentiality and Integrity | A large attack surface, particularly those that are exposed on the internet, will increase the probability of a compromise. | Minimize the application attack surface by limiting publicly exposed services. <ul><li>Use strong network controls by using Azure Virtual Networks, Network Security Groups (NSG), or Private Endpoints to protect against unsolicited traffic.</li><li>Use Azure Private Endpoints to block all internet connections to services that do not need to be publicly exposed.</li></ul>
| 6 | Integrity | Exploitation of insufficient logging and monitoring is the bedrock of nearly every major incident. Attackers rely on the lack of monitoring and timely response to achieve their goals without being detected. |  Logging of critical application events must be performed to ensure that, should a security incident occur, incident response and root-cause analysis may be done. Steps must also be taken to ensure that logs are available and cannot be overwritten or destroyed through malicious or accidental occurrences. At a minimum, the following events should be logged. <ul><li>Login/logout events.</li><li>Password change events (if not integrated into SSO).</li><li> Privilege delegation events.</li><li>Security validation failures (e.g., input validation or authorization check failures).</li><li>Application errors and system events.</li><li>Application and system startups and shutdowns, as well as logging initialization.</li></ul>
| 7 | Confidentiality and Integrity |Secrets leaking into unsecured locations are an easy way for adversaries to gain access to a system. These secrets can be used to either spoof the owners of these secrets or, in the case of encryption keys, use them to decrypt data. | Proper storage and management of secrets is critical in protecting systems from compromises, in most cases, with severe impact. <ul><li>Never store secrets in code or configuration files. Instead, use a vault or any secure container (such as encrypted variables) to store secrets.</li><li>Separate application secrets by environment.</li><li>Rotate all secrets before turning over the application to the customer.</li></ul>
| 8 | Confidentiality and Integrity | CI/CD Pipelines should always be cleansed of sensitive information being leaked. |  <ul><li>All secrets, key materials, and credentials stored in CI/CD Pipelines should always be cleansed of sensitive information that can be witnessed in plaintext.</li><li>Add static code analysis tools in CI/CD pipelines to gain security insights about the code being developed, gate insecure code from entering production, and generate an overall more secure and complete solution.</li><li>Add dependency scanning to CI/CD pipelines to ensure you are using the most current and secure libraries.</li><li>Add credential scanning to CI/CD pipelines to ensure secrets, key information, and passwords are not leaked into the open.</li></ul>
| 9 |  Availability  |  Distributed denial of service (DDoS) attacks is some of the largest availability and security concerns facing customers that are moving their applications to the cloud. A DDoS attempts to exhaust an application's resources, making the application unavailable to legitimate users. | DDoS protection is important to ensure the availability of your cloud services. Azure provides DDoS solutions to you and your customers to provide extra safeguards to this popular attack.

---

## Appendix

### Security Principles
  
---
  
#### Confidentiality 
Confidentiality refers to the objective of keeping data private or secret. In practice, it’s about controlling access to data to prevent unauthorized disclosure.

 #### Integrity
Integrity is about ensuring that data has not been tampered with and, therefore, can be trusted. It is correct, authentic, and reliable.

  #### Avaliability
Availability means that networks, systems, and applications are up and running. It ensures that authorized users have timely, reliable access to resources when they are needed.

### Microsoft Zero Trust Principles
  
---

#### Verify explicitly  
  
Always authenticate and authorize based on all available data points, including user identity, location, device health, service or workload, data classification, and anomalies.

#### Use least privileged access
  
Limit user access with just-in-time and just-enough-access (JIT/JEA), risk-based adaptive policies, and data protection to help secure both data and productivity.

#### Assume breach  
  
Minimize blast radius and segment access. Verify end-to-end encryption and use analytics to get visibility, drive threat detection, and improve defenses.

### Commercial Data Classification Reference
  
---
|  Classification | Guidelines for Classification |
|-----------------|-------------------------------|
|    Sensitive        |  Data that is to have the most limited access and requires a high degree of integrity. This is typically data that will do the most damage to the organization should it be disclosed. Personal data (including PII) falls into this category and includes any identifier, such as name, an identification number, location data, online identifier. This also includes data related to one or more factors specific to the physical, psychological, genetic, mental, economic, cultural, or social identity of an individual. |
|   Confidential      |  Data that might be less restrictive within the company but might cause damage if disclosed. |
|   Private           |  Private data is usually compartmental data that might not do the company damage but must be keep private for other reasons. Human resources data is one example of data that can be classified as private. |
|   Proprietary       |  Proprietary data is data that is disclosed outside the company on a limited basis or contains information that could reduce the company's competitive advantage, such as the technical specifications of a new product. |
|   Public            |  Public data is the least sensitive data used by the company and would cause the least harm if disclosed. This could be anything from data used for marketing to the number of employees in the company. |
