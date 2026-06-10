---
title: "Using Microsoft Purview DSPM to Discover Sensitive Data in SharePoint with Loose Security"
date: 2026-06-10T10:00:00-05:00
draft: false
image: "purview-dspm-sharepoint-discovery.png"
---

For small and medium-sized businesses (SMBs), SharePoint has become the de facto hub for content collaboration. Teams create sites, share documents, and manage projects—often without realizing they've left sensitive customer data, financial records, or proprietary information exposed to the wrong people. The problem isn't malice; it's complexity. SharePoint's permission model is powerful but unintuitive, and sprawling team sites accumulate governance debt faster than anyone can manually audit.

Microsoft Purview Data Security Posture Management (DSPM) changes the equation. Instead of waiting for a breach to discover oversharing, DSPM continuously scans your SharePoint environment, identifies sensitive data buried in that mess, flags risky access patterns, and guides you through remediation—all without requiring a dedicated security team.

This post walks through how DSPM works, the SharePoint security gaps it uncovers, and how to use it to reclaim visibility and control over your data.

## What Is Purview DSPM?

Purview DSPM is Microsoft's unified data security control plane. Think of it as a persistent spotlight scanning your entire Microsoft 365 environment, answering four critical questions:

1. **What sensitive data exists** in your organization?
2. **Where does it live** (SharePoint, OneDrive, Teams, Exchange, third-party SaaS)?
3. **Who can access it** (anonymous users, the entire organization, external partners)?
4. **How well is it protected** by policies, labels, or controls?

DSPM goes beyond simple alerts. It consolidates insights from four Purview solutions—Data Loss Prevention (DLP), Insider Risk Management, Information Protection, and Data Security Investigations—into a unified dashboard. More importantly, it uses AI agents to analyze access patterns, spot policy gaps, and surface only the highest-priority risks. No alert fatigue. No security theater.

## The SharePoint Security Problem DSPM Solves

Most SMBs discover their SharePoint security gaps the hard way. Here are the patterns DSPM is built to catch:

### Anonymous (Anyone) Links

The most widespread failure point. A user creates a "shareable link" that grants access to literally anyone with the URL. The link persists indefinitely. No audit trail. External users can forward the link, download files, and bypass authentication entirely. DSPM flags these immediately.

### Overly Broad Organizational Sharing

Files shared with "Everyone in the organization" sound reasonable—until contractors, vendors, guests, and former employees are also "in the organization." Sensitive HR records, financial projections, or client data suddenly becomes visible to hundreds of people who shouldn't have access.

### External Sharing Without Expiration

You grant a consultant 30 days of access to a sensitive proposal. Six months later, that link still works—the consultant is long gone, but the link persists. DSPM identifies sharing links with no expiration date, flagging which ones are most at risk.

### Over-Provisioned Access

A developer needs read access to a specific project folder. Instead, they get site owner permissions because "it's easier than managing granular permissions." Now they can delete content, manage access, and see everything on the site. DSPM surfaces these access gaps.

### SharePoint Sprawl and Ungoverned Sites

Teams create sites freely, but no one reviews access after the project ends. Contractors leave. Departments reorganize. Old sites accumulate sensitive content with stale permissions. DSPM identifies the top-risk sites and scans them automatically every week.

### Missing Sensitivity Labels

Files lacking classification labels are invisible to DLP policies. An unlabeled spreadsheet containing customer payment data sits unprotected. DSPM discovers it and recommends labeling.

## How DSPM Discovers Sensitive Data

DSPM uses a combination of pattern matching and AI to identify sensitive information:

### Sensitive Information Types (SITs)

Purview includes over 300 built-in Sensitive Information Types. These detect:

- **PII:** Social Security Numbers, driver's licenses, passports, addresses
- **Financial:** Credit card numbers, bank account numbers, SWIFT codes
- **Health:** Patient health information, medical terms, drug names
- **Credentials:** API keys, connection strings, passwords
- **Country-specific identifiers:** UK National Insurance numbers, EU passport formats, and more

### Pattern Recognition + Context

Detection isn't just regex. Each SIT uses a primary pattern (the sensitive data itself) plus supporting elements. For example, detecting a credit card number is stronger if it appears near keywords like "card," "Visa," or "expiration date" within 300 characters. This proximity-based approach reduces false positives.

### Named Entity Recognition

DSPM uses AI-based entity recognition to detect person names, addresses, and medical terms—not just pattern matches.

### Custom SITs

Your organization can create custom sensitive information types for proprietary formats (employee IDs, product codes) that built-in types don't cover.

## DSPM's Data Risk Assessments: Automated Weekly Scanning

Once configured, DSPM runs automatic weekly assessments of your top 100 SharePoint sites by usage. Each scan:

- Checks up to 200,000 items per location
- Reports the total number of items containing sensitive data
- Flags how many are accessible via anonymous or external links
- Ranks sites by risk

No manual setup required per site. No ticket backlogs. Just continuous visibility.

Admins can also trigger on-demand classification scans for specific sites that contain unscanned items.

## Understanding DSPM Risk: The Four Security Objectives

DSPM doesn't assign numeric risk scores per file. Instead, it organizes remediation around four Security Objectives—each one addressing a specific threat:

### 1. Prevent Oversharing of Sensitive Data

This objective surfaces:
- **Total overshared items:** Count of sensitive files shared with "Anyone," "Everyone," external users, or too many people
- **Sharing breakdown:** How many items are overshared via anonymous links vs. external sharing vs. organizational-wide sharing
- **Site rankings:** Which sites have the highest concentration of risky sharing
- **30-day trends:** Is oversharing improving or getting worse?

### 2. Prevent Data Exposure in Copilot and AI Interactions

With Microsoft Copilot now summarizing and analyzing content across Microsoft 365, overshared or unprotected data can be rapidly surfaced and synthesized by AI. This objective tracks:
- Which AI agents in your tenant are accessing sensitive data
- How many agents are flagged as high-risk
- Coverage gaps where sensitive content isn't protected from Copilot access

### 3. Prevent Exfiltration to Risky Destinations

Sensitive files shared with personal email addresses, consumer apps, or external cloud storage are a common precursor to data breaches. This objective identifies:
- How many sensitive files are shared with external services
- Trend data showing whether exfiltration risk is increasing
- AI-powered investigations identifying recently exfiltrated sensitive data

### 4. Discover Sensitive Data in the Organization

The foundational objective surfaces:
- Total count of sensitive items discovered across Microsoft 365
- Data type breakdown (PII, financial, health, credentials)
- Coverage gaps where sensitive data isn't labeled or protected

## Remediation: From Discovery to Action

DSPM doesn't just report problems—it guides you through fixing them. Remediation is built into each Security Objective as a step-by-step workflow.

### Item-Level Actions (SharePoint Files)

For each overshared file, you can:

- **Resolve as false positive:** If the AI was wrong, mark it and move on
- **Apply a sensitivity label:** Label unlabeled or mislabeled items in bulk
- **Notify the site owner:** Send a templated email alerting the owner to fix access
- **Remove the sharing link:** Revoke the external or anonymous link (the owner then establishes a more restrictive link)
- **Bulk removal:** Select multiple risky links across sites and remove them in one operation

### Policy-Level Automation

From within DSPM, you can create DLP policies, auto-labeling policies, and information barriers without leaving the interface:

- **Restrict Copilot access by label:** Prevent Copilot from summarizing content with specific sensitivity labels
- **Auto-labeling:** Automatically apply labels to newly detected sensitive files
- **Retention policies:** Auto-delete content not accessed for 3+ years

### AI-Powered Automated Remediation

Under your supervision, DSPM's AI agents can:
- Remove public sharing links automatically
- Apply DLP policies to risky assets
- Revoke over-provisioned permissions

All actions are logged and auditable.

## Why DSPM Matters for SMBs

For small and medium-sized businesses, DSPM addresses a hard problem: **how to maintain data governance without hiring a dedicated compliance officer.**

### Cost Reality

- **Microsoft 365 Business Premium (base):** ~$22/user/month
- **Purview Suite add-on (includes DSPM):** ~$10/user/month
- **Total:** ~$32/user/month

For comparison, a full Microsoft 365 E5 license (which includes DSPM) costs ~$57/user/month. The Business Premium + Purview Suite path gives you compliance capability at 40% lower cost.

### Practical Benefits

1. **Automated discovery:** Weekly scans of your top 100 SharePoint sites run without manual effort
2. **Guided remediation:** Actionable workflows replace overwhelming alert queues
3. **No specialist required:** DSPM surfaces only high-priority risks, making it manageable for IT teams without dedicated security staff
4. **Continuous improvement:** 30-day trend graphs let you demonstrate progress to leadership
5. **Regulatory alignment:** Built-in SIT coverage for HIPAA, PCI-DSS, GDPR, and other frameworks

### The ROI Calculation

The cost of a single data breach—regulatory fines, customer notification, reputation damage, remediation—far exceeds the annual cost of DSPM licensing. For SMBs handling customer data, payment information, or health records, DSPM is a cost-effective insurance policy.

## Getting Started: A Practical Workflow

If your organization is ready to implement DSPM, here's how to proceed:

### Step 1: Enable DSPM

DSPM is available to organizations with:
- Microsoft 365 Business Premium + Purview Suite add-on, OR
- Microsoft 365 E3/E5 with Compliance or Security licenses

Verify your licensing in the Microsoft 365 admin center.

### Step 2: Configure Initial Scan

Log into the Purview Compliance Portal, navigate to Data Security Posture Management, and enable the automated weekly assessment. DSPM will scan your top 100 SharePoint sites by default.

For item-level remediation (the most granular control), register an Entra app per Microsoft's documentation. This requires admin credentials but is a one-time setup.

### Step 3: Review the Posture Dashboard

The Posture page displays all four Security Objectives. Start with **Oversharing Prevention**—this is where most SMBs find the biggest quick wins.

### Step 4: Triage High-Risk Sites

Identify the top 5-10 overshared sites. Review each one for:
- Anonymous links that can be revoked immediately
- External sharing without expiration
- Sensitive content that should be labeled

### Step 5: Create Your First DLP Policy

Use the guided workflow to create a DLP policy based on DSPM findings. A good first policy: block DLP rule violations when someone tries to share files labeled "Confidential" with external users.

### Step 6: Monitor Trends

Return to the Posture dashboard monthly. Watch the 30-day trend graphs. Over time, you should see improvements in coverage, reductions in risky sharing, and better compliance posture.

## Integration with the Broader Microsoft 365 Ecosystem

DSPM doesn't operate in isolation. It integrates seamlessly with:

- **Data Loss Prevention (DLP):** Create policies directly from DSPM findings
- **Sensitivity Labels:** Identify unlabeled content and drive auto-labeling policies
- **Insider Risk Management:** Combine DLP + IRM policies to catch exfiltration patterns
- **Microsoft Security Copilot:** Accelerate analysis and remediation prioritization
- **Microsoft Defender XDR:** Correlate data risks with identity and endpoint risks
- **SharePoint Admin Center:** Link to detailed access governance reports
- **Third-party SaaS via partner solutions:** Extend DSPM coverage to Salesforce, Snowflake, Databricks, Google Cloud Platform, and others

This integrated approach means you're not managing disparate security tools—you're operating a cohesive, AI-powered data security strategy.

## Key Takeaways

1. **DSPM converts invisible data exposure into visible, actionable findings.** You can't fix what you can't see.

2. **Automated weekly scans eliminate the need for manual audits.** For SMBs, this is a game-changer.

3. **Guided remediation workflows replace overwhelming alert queues.** Item-level actions, policy creation, and auto-labeling all happen within a single interface.

4. **AI agents reduce manual toil.** Under your supervision, automated remediation saves time and reduces human error.

5. **Cost-effective compliance.** Purview Suite for Business Premium provides SMB-friendly pricing without sacrificing functionality.

6. **SharePoint sprawl is solvable.** DSPM's continuous scanning brings governance to even the messiest SharePoint environments.

The sensitive data in your SharePoint sites isn't going away. But with DSPM, neither is your visibility into it. Start with a pilot assessment on your top 10 sites, and let the data guide your remediation priorities.

---

**For more on SMB security strategy, governance, and data protection, explore our [resource library](../../) and [books on AI-enabled blue teams and infrastructure security](../../books/).**
