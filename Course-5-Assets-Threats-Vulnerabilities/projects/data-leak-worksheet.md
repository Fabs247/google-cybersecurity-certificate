# Data Leak Worksheet — Information Privacy Analysis

## Project Description

In this project, I analyzed a real-world data leak scenario at an educational technology company and evaluated the effectiveness of existing data privacy controls. Using the NIST Cybersecurity Framework (CSF) and NIST Special Publication 800-53 (SP 800-53) Access Control guideline AC-6 as reference frameworks, I identified the root causes of a confidential data leak, reviewed current least privilege controls, recommended specific control enhancements, and justified how those improvements would prevent similar incidents in the future. This exercise directly mirrors the work performed by GRC analysts, privacy officers, and security consultants when conducting post-incident reviews and strengthening information security policies.

---

## Incident Summary

A sales manager shared access to a folder of internal-only documents with their team during a meeting. The folder contained files associated with a new unannounced product, including customer analytics and promotional materials. After the meeting, the manager did not revoke access to the internal folder but warned the team to wait for approval before sharing the promotional materials with others.

During a video call with a business partner, a member of the sales team forgot the manager's warning. The sales representative intended to share a link only to the promotional materials so the business partner could circulate them to customers. However, the representative accidentally shared a link to the entire internal folder instead. The business partner then posted the link on their company's social media page, assuming it contained only the promotional materials  resulting in a public exposure of confidential internal business plans.

---

## Data Leak Worksheet

| Control | Least Privilege |
|---------|----------------|
| **Issue(s)** | The data leak was caused by two compounding failures of the principle of least privilege. First, the manager granted the sales representative access to the entire internal folder rather than only the specific promotional materials needed for the sales call,violating the need-to-know principle. Second, the manager failed to revoke folder access after the meeting, leaving sensitive internal documents unnecessarily accessible. These oversights combined with human error during the sales call resulted in confidential business plans being publicly exposed on social media. |
| **Review** | NIST SP 800-53 AC-6 (Least Privilege) establishes that users should only be granted the minimal level of access and authorization required to complete their specific task or function nothing more. The control specifies that processes, user accounts, and roles must be enforced to prevent users from operating at privilege levels higher than necessary to accomplish legitimate business objectives. AC-6 provides control enhancements including role-based access restrictions, automatic access revocation after a defined time period, activity logging of provisioned accounts, and regular audits of user privileges to maintain least privilege over time. |
| **Recommendation(s)** | **1. Restrict access to sensitive resources based on user role:** Implement role-based access controls (RBAC) so that employees only receive access to the specific files and folders required for their job function. The sales representative should have been given access only to the promotional materials folder not the entire internal documents repository. Folder-level permissions should be configured to prevent accidental sharing of entire directories containing mixed-sensitivity content. **2. Automatically revoke access to information after a period of time:** Implement time-limited access controls so that shared folder permissions automatically expire after a defined period, for example 24 hours after a meeting concludes. This would have prevented the sales representative from retaining access to internal documents after the meeting and eliminated the possibility of accidentally sharing them during a later sales call. |
| **Justification** | Implementing role-based access restrictions directly addresses the root cause of this incident by ensuring employees can only access the specific resources their role requires making it technically impossible for a sales representative to share a link to confidential internal documents they should never have had access to in the first place. Automatic access revocation addresses the secondary failure by eliminating the risk of human error in access management removing the dependency on managers remembering to manually revoke permissions after meetings. Together these two enhancements create a layered least privilege enforcement system that reduces both the likelihood and the potential impact of similar data leak incidents in the future. |

---

## Security Plan Snapshot

| Function | Category | Subcategory | Reference |
|----------|----------|-------------|-----------|
| **Protect** | PR.DS: Data Security | PR.DS-5: Protections against data leaks | NIST SP 800-53: AC-6 |

---

## NIST SP 800-53: AC-6 Reference

| AC-6 | Least Privilege |
|------|----------------|
| **Control** | Only the minimal access and authorization required to complete a task or function should be provided to users. |
| **Discussion** | Processes, user accounts, and roles should be enforced as necessary to achieve least privilege. The intention is to prevent a user from operating at privilege levels higher than what is necessary to accomplish business objectives. |
| **Control Enhancements Applied** | 1. Restrict access to sensitive resources based on user role. 2. Automatically revoke access to information after a period of time. |

---

## Root Cause Analysis

### What went wrong — step by step

```
Step 1 — Manager granted EXCESSIVE access
→ Sales representative received access to
  the ENTIRE internal folder
→ Should have received access to ONLY
  the promotional materials
→ Violation of least privilege principle ❌

Step 2 — Manager FAILED to revoke access
→ After the meeting access was not removed
→ Representative retained unnecessary access
→ Violation of access management policy ❌

Step 3 — Human error during sales call
→ Representative forgot manager's warning
→ Accidentally shared entire folder link
→ Instead of promotional materials only
→ Human error amplified by excessive access ❌

Step 4 — Business partner posted publicly
→ Assumed link was promotional materials
→ Posted to company social media page
→ Internal business plans publicly exposed ❌
→ Data leak confirmed
```

---

## Contributing Factors

| Factor | Description | Control Failure |
|--------|-------------|----------------|
| Excessive access granted | Manager shared entire folder instead of specific files | Least privilege not enforced |
| Access not revoked | Manager forgot to remove permissions after meeting | No automatic revocation policy |
| Human error | Representative accidentally shared wrong link | No technical controls preventing this |
| No access monitoring | No alerts for unusual sharing activity | Lack of activity logging |
| Inadequate training | Representative forgot manager's warning | Security awareness gap |

---

## Recommended Control Enhancements

### Enhancement 1 — Role-Based Access Control (RBAC)

```
WHAT:
Restrict access to sensitive resources
based on user role and need-to-know

HOW TO IMPLEMENT:
→ Configure folder-level permissions
  so employees only see files their
  role requires
→ Sales representatives get access to:
  ✅ Promotional materials
  ❌ Customer analytics (restricted)
  ❌ Internal business plans (restricted)
→ Managers approve access requests
  for specific files only
→ Never share entire folder access
  when only one file is needed

PREVENTS:
→ Employees accessing more than they need
→ Accidental sharing of sensitive content
→ Insider threat from excessive access
```

---

### Enhancement 2 — Automatic Access Revocation

```
WHAT:
Automatically revoke access to shared
information after a defined time period

HOW TO IMPLEMENT:
→ Set expiration on all shared folder links
  (example: 24 hours after meeting)
→ Send reminder to manager when access
  is about to expire
→ Require explicit renewal for continued access
→ Implement DLP (Data Loss Prevention) tools
  that flag and block sharing of restricted
  content outside approved channels

PREVENTS:
→ Forgotten permissions lingering indefinitely
→ Human error in manual access management
→ Long-term exposure from one-time shares
→ Reduces window of opportunity for leaks
```

---

## Additional Recommendations

```
Beyond the two primary AC-6 enhancements:

3. Activity logging of provisioned accounts
→ Log every file access and sharing event
→ Alert security team when internal documents
  are shared externally
→ Create audit trail for investigations

4. Regular privilege audits
→ Quarterly review of all user access levels
→ Remove any unnecessary permissions found
→ Ensure role-based access remains current
  as employees change roles

5. Security awareness training
→ Train employees on data classification
→ Emphasize consequences of accidental leaks
→ Teach proper file sharing procedures
→ Phishing and social engineering awareness
```

---

## NIST CSF Alignment

```
IDENTIFY:
→ Identified sensitive data assets
  (internal business plans, customer analytics)
→ Assessed data handling risks
→ Documented access control gaps

PROTECT:
→ Recommended RBAC implementation
→ Recommended automatic access revocation
→ Aligned with NIST SP 800-53 AC-6
→ Principle of least privilege enforced

DETECT:
→ Recommended activity logging
→ Flag unusual external sharing events
→ Monitor for data exfiltration patterns

RESPOND:
→ Audit underway to assess full impact
→ Immediate access revocation to leaked folder
→ Notification to affected stakeholders

RECOVER:
→ Remove publicly accessible link immediately
→ Assess reputational damage
→ Update data handling policies
→ Implement recommended controls
```

---

## Summary

This data leak incident resulted from two compounding failures of the principle of least privilege a manager granting excessive folder access to a sales representative and subsequently failing to revoke that access after the meeting concluded. These failures, combined with human error during a sales call, resulted in confidential internal business plans being publicly shared on social media. By implementing NIST SP 800-53 AC-6 control enhancements specifically role-based access restrictions and automatic time-limited access revocation, the company can create technical controls that enforce least privilege without relying on human memory or manual processes. These enhancements directly address the root causes of the incident and significantly reduce the likelihood of similar data leaks occurring in the future, protecting the company's intellectual property, customer data, and regulatory compliance standing.

---

## Skills Demonstrated

```
✅ Data privacy analysis and incident review
✅ NIST SP 800-53 AC-6 framework application
✅ Principle of least privilege evaluation
✅ Root cause analysis of security incidents
✅ Security control enhancement recommendations
✅ GRC policy and compliance thinking
✅ Risk-based security recommendation justification
✅ NIST CSF Protect function implementation
✅ Professional security documentation
```
