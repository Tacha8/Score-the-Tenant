# Investigating a Defender for Storage Malware Alert and an Exempted Security Recommendation

## Scenario

A Microsoft Defender for Storage alert identified a malicious file uploaded to an Azure Storage Account. The investigation focused on determining whether the affected storage account was covered by Defender for Storage, understanding how the malicious upload occurred, and tracing why an existing security recommendation had not been escalated.

The objective was to connect the security detection to the underlying cloud misconfiguration, investigate the associated policy exemption, and verify that the alert reached the unified security operations queue.

## Environment

- **Cloud platform:** Microsoft Azure
- **Security platform:** Microsoft Defender for Cloud
- **Workload protection:** Defender for Storage
- **Posture management:** Cloud Security Posture Management (CSPM)
- **Security operations:** Microsoft Defender XDR unified security portal
- **Policy management:** Azure Policy
- **Affected resource type:** Azure Storage Account
- **Environment:** Live multi-user Azure training tenant
- **Access level:** Azure Reader, Sentinel Reader, and Defender XDR portal permissions scoped to the training environment
- **Investigation tools:** Azure Portal, Defender for Cloud, Security Recommendations, Security Alerts, Exemptions, and Microsoft Defender portal

## Investigation

### 1. Auditing Defender for Storage coverage

I began by reviewing the storage accounts in the assigned resource group to determine which resources had Defender for Storage enabled.

The resource group contained five storage accounts. I inspected the Defender for Cloud protection status at the individual storage-account level rather than assuming that subscription-level configuration guaranteed coverage across every resource.

The investigation identified that four of the five storage accounts were protected, while one account did not have Defender for Storage enabled.

The unprotected account represented a monitoring gap because security controls configured at the subscription level do not necessarily guarantee that every individual resource is covered.

<img width="958" height="516" alt="image" src="https://github.com/user-attachments/assets/7f7ff9d5-a5e4-426e-b909-d341da7c1409" />


**Security takeaway:** Defender for Cloud coverage must be validated at the resource level, particularly when new resources are created after the last security review.

### 2. Investigating the malware detection

I navigated to Microsoft Defender for Cloud's Security Alerts and located the alert titled:

`Malicious blob uploaded to storage account`

The alert was associated with the storage account identified in the lab scenario.

I reviewed the alert details to understand what file triggered the detection, where it was uploaded, when the activity occurred, and what response Defender performed.

The lab also demonstrated that Defender for Storage malware scanning can stamp the scan verdict onto the affected blob through its index tags.

The relevant index tag was:

`Malware scanning scan result: Malicious`

<img width="957" height="637" alt="image" src="https://github.com/user-attachments/assets/3f8c4e7d-f535-4b3b-8dbd-cc2c1e69f7d3" />


This provides an additional source of evidence on the storage object itself, rather than relying exclusively on the security alert.

**Security takeaway:** Malware detection should be investigated using both the alert details and the affected storage object's available metadata.

### 3. Investigating the security recommendation

After reviewing the malware alert, I investigated the underlying storage-account configuration to understand how the malicious upload could have occurred.

The lab scenario identified two relevant conditions:

- The storage account allowed public blob access.
- A leaked Shared Access Signature (SAS) token was used in the malicious upload scenario.

I then reviewed Defender for Cloud's Security Recommendations and located the recommendation:

`Storage account public access should be disallowed`

This recommendation identified a storage configuration that increased the account's exposure.

The lab required clearing the recommendation status filter because recommendations marked as Not evaluated, including exempted recommendations, were hidden from the default view.

This demonstrated why analysts should review recommendation status filters rather than assuming that a missing recommendation means the resource has no security findings.

**Security takeaway:** CSPM recommendations can identify risky configurations before an incident occurs, making them an important preventative security control.

### 4. Investigating the policy exemption

I then investigated why the storage-account recommendation had not remained visible as an active security concern.

The lab scenario revealed that the recommendation had been exempted.

I reviewed the recommendation's Not applicable section and followed the exemption details to the associated Azure Policy exemption object.

This demonstrated how a security recommendation can be removed from the active remediation workflow through a human-approved exemption.

Although exemptions can be legitimate when risks are mitigated elsewhere or formally accepted, they can also allow security weaknesses to remain unresolved.

The investigation highlighted the importance of reviewing exemption scope, justification, ownership, and expiration.

**Security takeaway:** A security exemption is not proof that a vulnerability or misconfiguration has been fixed. It represents an accepted or otherwise exempted risk that may still require monitoring.

### 5. Verifying the alert in the unified security portal

For the final stage, I opened the Microsoft Defender portal at `security.microsoft.com` and located the incident associated with the Defender for Storage malware alert.

I reviewed the incident details, including its title, severity, associated alert, entities, and service source.

This step demonstrated how Defender for Cloud detections can flow into the unified security operations experience used by SOC analysts.

The unified queue gives analysts a centralized place to review security incidents across Microsoft's security products.

**Security takeaway:** Generating a detection is only part of the security workflow. The alert must also reach the appropriate monitoring and incident-response process.

## What broke / what surprised me

- Subscription-level Defender configuration does not guarantee that every individual storage account is protected.
- The newest storage account was the coverage gap, demonstrating how newly created resources can fall outside established security reviews.
- A security recommendation existed for the public-access misconfiguration, but its exempted status caused it to disappear from the default recommendation view.
- A human-approved policy exemption can allow a security weakness to remain in place even when Defender for Cloud has already identified it.
- The malware detection and posture recommendation represented two different stages of security: one detected an actual malicious upload, while the other identified a configuration that increased exposure before the incident.
- The investigation reinforced that analysts must examine the configuration and governance decisions behind an alert, not simply close the incident after identifying the malicious file.

## Findings and recommendations

### Findings

1. Four of the five storage accounts in the resource group had Defender for Storage protection enabled.
2. One storage account was not protected, representing a Defender coverage gap.
3. Defender for Storage detected a malicious blob uploaded to the affected storage account.
4. The affected account allowed public blob access.
5. The lab scenario identified a leaked SAS token as the mechanism associated with the malicious upload.
6. Defender for Cloud had an existing recommendation identifying the public-access configuration as a security concern.
7. The recommendation had been exempted through an Azure Policy exemption.
8. The Defender for Storage alert was also available through the unified Microsoft Defender security portal.

### Recommendations

1. **Enforce Defender for Storage coverage:** Audit storage accounts regularly and ensure newly created resources receive the required Defender for Storage protection. Consider using Azure Policy and governance controls to prevent coverage gaps.
2. **Reduce storage exposure and secure SAS tokens:** Disable unnecessary public blob access, use narrowly scoped SAS permissions and expiration periods, and review token issuance and exposure. Prefer Microsoft Entra ID-based authentication where appropriate.
3. **Govern security exemptions:** Require documented business justification, an accountable owner, an expiration or review date, and periodic reassessment for Defender for Cloud recommendation exemptions.
4. **Maintain detection and response visibility:** Verify that Defender for Storage alerts reach the unified security operations queue and that incidents are reviewed and handled through an established response process.

## What I learned

- **Technical:** Defender for Storage provides workload protection through malware scanning and security alerts, while Defender for Cloud CSPM identifies security misconfigurations and posture weaknesses.
- **Cloud governance:** Defender coverage must be reviewed at the individual resource level because subscription-level enablement does not guarantee universal protection.
- **Azure Policy:** Policy exemptions can suppress security recommendations without actually correcting the underlying configuration.
- **Security operations:** Analysts should correlate security alerts with posture recommendations, resource configurations, and governance decisions to understand why an incident was possible.
- **What I would do differently:** I would establish a recurring coverage audit, review outstanding exemptions, and capture evidence of the storage configuration, malware alert, recommendation, exemption object, and unified incident during the investigation.

---

**Project classification:** Azure cloud security investigation using Microsoft Defender for Cloud, Defender for Storage, Azure Policy, and Microsoft Defender XDR in a live multi-user Azure training tenant.
