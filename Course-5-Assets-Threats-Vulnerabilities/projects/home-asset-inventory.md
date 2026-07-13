# Home Asset Inventory — Network Asset Classification

## Project Description

In this project, I conducted a home network asset inventory and classification exercise as part of an asset management security assessment. Asset management is a foundational component of any organization's security strategy — you cannot protect what you do not know exists. I identified network-connected devices, documented their key characteristics including network access frequency, ownership, and physical location, and classified each asset based on its sensitivity level and potential business impact if compromised. This exercise directly mirrors the asset management processes performed by security analysts in enterprise environments to identify, track, and prioritize the protection of organizational assets.

---

## Asset Inventory Table

| # | Asset | Network Access | Owner | Location | Notes | Sensitivity |
|---|-------|---------------|-------|----------|-------|-------------|
| 1 | Network Router | Continuous | Internet Service Provider (ISP) | On-premises | Has a 2.4 GHz and 5 GHz connection. All devices on the home network connect to the 5 GHz frequency. Acts as the gateway between the home network and the internet — compromise of this device would expose all connected assets. | Confidential |
| 2 | Desktop Computer | Occasional | Homeowner | On-premises | Contains private and sensitive information including personal photos, financial documents, and business files. Only the homeowner should have access. Stores locally saved passwords and browser history. | Restricted |
| 3 | Guest Smartphone | Occasional | Friend/Visitor | On and Off-premises | Connects to the home network via guest access. Owner is not always careful about device security may have unpatched vulnerabilities. Does not store business data but provides a potential entry point to the network. | Internal-only |
| 4 | External Hard Drive | Occasional | Homeowner | On-premises | Contains backup copies of all business files, financial records, and sensitive client data. Connects via USB,not always connected to the network but stores the most sensitive data on the premises. Physical theft is a primary risk. | Restricted |
| 5 | Smart Speaker | Continuous | Homeowner | On-premises | Always connected to the home network and the internet. Contains voice recordings and is linked to personal accounts. Could be exploited to eavesdrop on confidential business conversations. Firmware updates are infrequent. | Confidential |
| 6 | Laptop Computer | Occasional | Homeowner | On and Off-premises | Used for remote business operations and frequently connects to external networks including public Wi-Fi. Contains business emails, client communications, and financial software. Higher risk due to mobility and external network exposure. | Restricted |

---

## Sensitivity Classification Guide

| Category | Access Designation | Description |
|----------|-------------------|-------------|
| **Restricted** | Need-to-know | Highest sensitivity contains business-critical or personal data. Unauthorized access could cause significant harm. |
| **Confidential** | Limited to specific users | Sensitive data with controlled access. Only authorized individuals may access. |
| **Internal-only** | Users on-premises | Data accessible to approved users on the network but not publicly available. |
| **Public** | Anyone | No sensitive data. Freely available to all users without restriction. |

---

## Asset Analysis

### High Priority Assets — Restricted

**Desktop Computer and External Hard Drive** are classified as **Restricted** because they contain the most sensitive business and personal data on the network. The desktop stores financial documents, client files, and personal photos that should only be accessible to the homeowner. The external hard drive contains backup copies of all critical business data — if stolen or compromised, this data could expose the entire business operation. Both require strong access controls, encryption, and physical security measures.

**Laptop Computer** is also classified as **Restricted** due to its mobility. Unlike the desktop, the laptop travels outside the home network and frequently connects to untrusted public Wi-Fi networks. This significantly increases its attack surface. A compromised laptop could provide an attacker with direct access to business emails, financial software, and client communications while also serving as an entry point back into the home network through VPN connections.

---

### Medium Priority Assets — Confidential

**Network Router** is classified as **Confidential** because it controls all traffic entering and leaving the home network. If an attacker gains access to the router, they can intercept all network communications, redirect traffic, and access every device on the network. The ISP owns the router, which limits direct control over firmware updates and security configurations — making monitoring and access restriction critical compensating controls.

**Smart Speaker** is classified as **Confidential** because it is always listening and always connected to the internet. It is linked to personal accounts including shopping, calendar, and potentially financial services. The continuous network connection and voice recording capability create significant confidentiality risks — particularly in a home business environment where sensitive conversations may be overheard and recorded.

---

### Lower Priority Assets — Internal-only

**Guest Smartphone** is classified as **Internal-only** because it belongs to a visitor rather than the homeowner and does not store business data. However, it still represents a network security risk because the device owner may not maintain proper security hygiene — including outdated software, weak passwords, or malware. Isolating guest devices on a separate network segment or VLAN would reduce this risk significantly.

---

## Security Recommendations

Based on the asset classification exercise the following security controls are recommended:

| Asset | Risk | Recommendation |
|-------|------|----------------|
| Network Router | Gateway compromise exposes all devices | Change default credentials, enable WPA3 encryption, create separate guest network, update firmware regularly |
| Desktop Computer | Contains sensitive personal and business data | Enable full disk encryption, implement strong password policy, enable automatic updates |
| External Hard Drive | Physical theft or unauthorized access | Enable encryption on all stored data, store in locked location when not in use, maintain offsite backup |
| Laptop Computer | Mobile device exposed to untrusted networks | Always use VPN on public Wi-Fi, enable full disk encryption, enable remote wipe capability |
| Smart Speaker | Always-on microphone and internet connection | Mute microphone when discussing sensitive business matters, review linked account permissions regularly |
| Guest Smartphone | Unknown security posture of device owner | Isolate on separate guest network with internet-only access, no access to internal network resources |

---

## Key Takeaways

This asset inventory exercise demonstrated several important security principles:

**Visibility is the foundation of security** — you cannot protect assets you do not know exist. Creating a comprehensive inventory revealed six devices on the home network, each with different risk profiles and protection requirements.

**Sensitivity classification drives prioritization** — not all assets require the same level of protection. By classifying each device, security efforts and resources can be directed toward the highest risk assets first, specifically the restricted-level devices containing business-critical data.

**Mobility increases risk** — the laptop computer carries a higher risk classification than the desktop despite storing similar data because its exposure to external networks significantly increases its attack surface.

**Third-party devices present uncontrolled risk** — the guest smartphone highlighted that devices owned by others connected to the network introduce risks outside the homeowner's direct control, reinforcing the importance of network segmentation.

**CIA Triad applied to asset classification:**
- **Confidentiality** — restricted assets contain data that must only be accessed by authorized users
- **Integrity** — business files on the external hard drive must not be altered by unauthorized parties
- **Availability** — the network router must remain operational to maintain connectivity for all business operations

---

## Summary

This project involved creating a structured home network asset inventory by identifying six network-connected devices, documenting their characteristics, and classifying each according to its sensitivity level. Three assets were classified as Restricted due to their storage of sensitive business and personal data, two as Confidential due to their network-level access and always-on connectivity, and one as Internal-only due to its limited access and third-party ownership. The exercise reinforced that effective asset management requires not only identifying what assets exist but understanding their risk profile, ownership, connectivity, and potential business impact if compromised — skills directly applicable to enterprise asset management, GRC compliance frameworks, and SOC analyst operations.
