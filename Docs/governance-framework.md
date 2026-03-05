# Enterprise BI Governance Framework

## Document Information

| Field | Detail |
|-------|--------|
| **Version** | 1.0 |
| **Last Updated** | March 2026 |
| **Owner** | BI Center of Excellence (CoE) |
| **Review Cycle** | Quarterly |
| **Applies To** | All Power BI & Microsoft Fabric workspaces |

---

## 1. Purpose

This framework establishes the standards, policies, and procedures that govern how business intelligence assets are created, managed, secured, and maintained across the organization. It ensures consistency, data trust, and scalability as the BI environment grows from a handful of reports to an enterprise-scale analytics platform.

Every team member who creates, modifies, or consumes BI content is expected to follow these guidelines.

---

## 2. Naming Conventions

Consistent naming eliminates confusion, makes assets searchable, and ensures that anyone browsing a workspace can immediately understand what they are looking at.

### 2.1 Standard Pattern

All BI assets follow the **DEPT-ENV-Type-Description** pattern:

```
[Department]-[Environment]-[AssetType]-[Description]
```

### 2.2 Naming Components

| Component | Allowed Values | Example |
|-----------|---------------|---------|
| **Department** | Finance, Sales, Marketing, Operations, HR, Supply, Executive, Shared | `Sales` |
| **Environment** | Dev, Test, Prod | `Prod` |
| **Asset Type** | SM (Semantic Model), RPT (Report), DASH (Dashboard), DF (Dataflow), PL (Pipeline) | `RPT` |
| **Description** | Brief, descriptive name using PascalCase | `RegionalPerformance` |

### 2.3 Examples

| Asset | Correct Name | Incorrect Name |
|-------|-------------|----------------|
| Production sales report | `Sales-Prod-RPT-RegionalPerformance` | `Sales Report v3 FINAL` |
| Dev semantic model | `Finance-Dev-SM-BudgetActuals` | `finance model (copy)` |
| Production dashboard | `Executive-Prod-DASH-CompanyKPIs` | `Dashboard2` |
| Dataflow | `Sales-Prod-DF-CustomerCleansing` | `dataflow 1` |
| Data pipeline | `Shared-Prod-PL-DailyIngest` | `my pipeline` |

### 2.4 Workspace Naming

Workspaces follow a simplified pattern: **DEPT-ENV**

| Workspace | Purpose |
|-----------|---------|
| `Sales-Dev` | Development and experimentation for sales team |
| `Sales-Test` | User acceptance testing for sales content |
| `Sales-Prod` | Production content visible to end users |
| `Shared-Prod` | Cross-departmental certified datasets |
| `Executive-Prod` | C-suite dashboards and strategic reports |

### 2.5 Rules

- No spaces in names. Use hyphens between components and PascalCase within the description.
- Never include version numbers, dates, or words like "final", "copy", or "old" in asset names. Use Git or workspace versioning instead.
- Acronyms are acceptable only if universally understood within the organization (e.g., KPI, YTD, MTD).

---

## 3. Data Certification & Endorsement

Certification builds trust. When a user sees a **Certified** badge on a dataset, they know it has been reviewed, tested, and approved as the single source of truth. Without certification, analysts waste time building on unreliable data, leading to conflicting numbers in meetings.

### 3.1 Endorsement Levels

| Level | Badge | Meaning | Who Can Apply |
|-------|-------|---------|---------------|
| **No Endorsement** | None | Exploratory, personal, or in-development content | Any creator |
| **Promoted** | ⭐ Promoted | Stable and useful, but not formally reviewed as the enterprise standard | Dataset owner or workspace admin |
| **Certified** | ✅ Certified | Officially reviewed, tested, and approved as the organizational source of truth | BI CoE only (after completing the checklist below) |

### 3.2 Certification Checklist

A semantic model must pass **all** of the following criteria before receiving Certified status:

| # | Requirement | Verification Method |
|---|------------|-------------------|
| 1 | Star schema design with proper relationships | Model review by BI CoE |
| 2 | All measures documented with descriptions | Verify Description field is populated for every measure |
| 3 | Row-level security configured and tested | Test each role using "View as Role" in Power BI |
| 4 | No direct query to source in production (import or Direct Lake only) | Check connection mode in dataset settings |
| 5 | Data refresh succeeds for 7 consecutive days without failure | Review refresh history |
| 6 | No broken relationships or orphaned tables | Run Best Practice Analyzer (Tabular Editor) |
| 7 | Naming conventions followed for all tables, columns, and measures | Manual review against Section 2 |
| 8 | Data validation: key metrics match source system totals within 1% tolerance | Cross-check 3 key measures against source |
| 9 | Performance: report pages load in under 5 seconds on a standard connection | Performance testing |
| 10 | Owner and steward identified and documented | Confirm in asset registry |

### 3.3 Certification Workflow

```
Creator submits certification request
        │
        ▼
BI CoE reviews against checklist (1-3 business days)
        │
        ├── All criteria met ──► Certified badge applied
        │                        Owner notified
        │                        Added to Certified Assets registry
        │
        └── Gaps found ──► Feedback sent to creator
                           Creator resolves issues
                           Re-submit for review
```

### 3.4 Certification Maintenance

- Certified assets are **re-reviewed quarterly** or whenever a major change is deployed.
- If a certified dataset has **3 or more consecutive refresh failures**, its Certified badge is automatically suspended until the issue is resolved and re-reviewed.
- The BI CoE maintains a **Certified Assets Registry** (shared Excel/SharePoint list) tracking every certified asset, its owner, certification date, and next review date.

---

## 4. Access Control Matrix

Access control ensures that users only see the data they are authorized to view. This project uses a combination of **workspace roles** (who can access the workspace) and **row-level security** (what data they can see within a dataset).

### 4.1 Workspace Role Assignments

| Workspace Role | Permissions | Typical Users |
|----------------|------------|---------------|
| **Admin** | Full control: manage membership, publish, delete, configure settings | BI CoE lead, IT admin |
| **Member** | Publish content, edit reports, create new items (cannot manage membership) | BI developers, data analysts |
| **Contributor** | Edit existing reports and dashboards (cannot publish semantic models) | Power users, report authors |
| **Viewer** | View reports and dashboards only (no editing, no export to Excel unless explicitly granted) | Business users, managers, executives |

### 4.2 Row-Level Security (RLS) Roles

These RLS roles are configured within the Semantic Model to filter data automatically based on the user's assigned role:

| RLS Role | Table Filtered | DAX Filter Expression | Data Visible | Assigned To |
|----------|---------------|----------------------|-------------|-------------|
| Regional Manager — West | DimSalesTerritory | `[Region] = "West"` | West region sales, customers, and territories only | West regional managers and their direct reports |
| Regional Manager — East | DimSalesTerritory | `[Region] = "East"` | East region sales, customers, and territories only | East regional managers and their direct reports |
| Regional Manager — Central | DimSalesTerritory | `[Region] = "Central"` | Central region sales, customers, and territories only | Central regional managers and their direct reports |
| Regional Manager — Northwest | DimSalesTerritory | `[Region] = "Northwest"` | Northwest region sales, customers, and territories only | Northwest regional managers and their direct reports |
| Executive (Full Access) | *(no filter applied)* | *(none)* | All regions, all data | VP of Sales and above, CFO, CEO, BI CoE |

### 4.3 Combined Access Example

Below is an example of how workspace roles and RLS combine for a single user:

| User | Workspace Role | RLS Role | Result |
|------|---------------|----------|--------|
| Jane (West Region Mgr) | Viewer in `Sales-Prod` | Regional Manager — West | Can view reports; all visuals automatically filter to West data only |
| Tom (BI Developer) | Member in `Sales-Dev`, Viewer in `Sales-Prod` | Executive (Full Access) | Can build in Dev, view everything in Prod for testing |
| Sara (CFO) | Viewer in `Executive-Prod` | Executive (Full Access) | Can view executive dashboards with all regional data |
| Mike (Marketing Analyst) | Contributor in `Marketing-Dev` | No role in Sales model | Cannot access Sales semantic model at all |

### 4.4 Access Request Process

1. User submits an access request through the IT service portal (or designated SharePoint form).
2. Request is routed to the **data steward** for the relevant workspace.
3. Data steward verifies business justification and confirms appropriate RLS role.
4. IT admin or workspace admin grants the workspace role and adds the user to the appropriate Azure AD security group mapped to the RLS role.
5. Access is logged in the **Access Audit Log** with requestor, approver, date, and role granted.

---

## 5. Change Management

Changes to production BI assets must follow a structured process to prevent errors, data discrepancies, and broken reports that erode user trust.

### 5.1 Change Categories

| Category | Examples | Process Required |
|----------|---------|-----------------|
| **Low Risk** | Formatting changes, adding a tooltip, fixing a typo in a title | Peer review → deploy to Test → deploy to Prod |
| **Medium Risk** | New report page, new visual, adding a filter, new DAX measure | Peer review → deploy to Test → UAT sign-off → deploy to Prod |
| **High Risk** | Changing the data model (new tables, modified relationships, altered measures), changing RLS, modifying data refresh logic | Full review by BI CoE → deploy to Test → UAT sign-off → stakeholder approval → deploy to Prod |

### 5.2 Deployment Pipeline Steps

All changes flow through the three-stage deployment pipeline: **DEV → TEST → PROD**.

```
┌─────────────────────────────────────────────────────────────────┐
│                    CHANGE MANAGEMENT WORKFLOW                    │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  1. DEVELOP (DEV Workspace)                                     │
│     • Make all changes in the Dev workspace                     │
│     • Test locally using sample data                            │
│     • Run Best Practice Analyzer checks                         │
│     • Self-review: verify naming conventions                    │
│     • Commit changes to Git repository                          │
│                                                                 │
│  2. PEER REVIEW                                                 │
│     • A second BI developer reviews the changes                 │
│     • Review checklist:                                         │
│       ☐ DAX measures return expected results                    │
│       ☐ No broken relationships                                │
│       ☐ Naming conventions followed                             │
│       ☐ No hard-coded values that should be parameters          │
│       ☐ RLS roles still function correctly                      │
│                                                                 │
│  3. DEPLOY TO TEST                                              │
│     • Use Fabric deployment pipeline to push to Test workspace  │
│     • Data refresh runs against Test data source                │
│     • Automated validation: row counts, key metric comparisons  │
│                                                                 │
│  4. USER ACCEPTANCE TESTING (UAT)                               │
│     • Business stakeholder(s) review in the Test workspace      │
│     • Verify numbers match their expectations                   │
│     • Confirm visuals and interactions work correctly            │
│     • Sign-off documented via email or approval form             │
│                                                                 │
│  5. DEPLOY TO PRODUCTION                                        │
│     • For High Risk: requires BI CoE lead approval              │
│     • Deploy via pipeline during off-peak hours (before 7 AM)   │
│     • Monitor first data refresh after deployment                │
│     • Notify affected users of changes via email/Teams          │
│                                                                 │
│  6. POST-DEPLOYMENT MONITORING (48 hours)                       │
│     • Monitor refresh success/failure                           │
│     • Check usage metrics for anomalies                         │
│     • Be available for quick rollback if issues found           │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 5.3 Rollback Policy

- Every deployment to Production creates an automatic snapshot in the deployment pipeline.
- If a critical issue is found within 48 hours of deployment, the BI CoE can **roll back** to the previous version using the pipeline's comparison and revert feature.
- Rollbacks do not require re-approval but must be logged with the reason for rollback.

### 5.4 Emergency Changes

In rare cases where a production issue requires an immediate fix (e.g., a broken refresh exposing stale data to executives before a board meeting):

1. The BI developer makes the fix directly in the Test workspace.
2. Deploys to Prod with verbal approval from the BI CoE lead.
3. A retrospective is held within 24 hours to document the issue, fix, and any process improvements needed.
4. The Dev workspace is updated to reflect the emergency change.

---

## 6. Data Refresh Schedule

Reliable, predictable data refreshes are essential for user trust. If a dashboard shows stale data without warning, users lose confidence in the entire BI platform.

### 6.1 Refresh Schedule by Dataset

| Dataset | Refresh Frequency | Scheduled Time | Timeout | Owner |
|---------|-------------------|----------------|---------|-------|
| `Shared-Prod-SM-FactSales` | Daily | 6:00 AM ET | 60 min | BI CoE |
| `Shared-Prod-SM-DimProduct` | Daily | 5:30 AM ET | 15 min | BI CoE |
| `Shared-Prod-SM-DimCustomer` | Daily | 5:30 AM ET | 15 min | BI CoE |
| `Shared-Prod-SM-DimDate` | Monthly (1st of month) | 5:00 AM ET | 5 min | BI CoE |
| `Shared-Prod-SM-DimGeography` | Monthly (1st of month) | 5:00 AM ET | 5 min | BI CoE |
| `Shared-Prod-SM-DimSalesTerritory` | Quarterly | 5:00 AM ET | 5 min | BI CoE |
| `Finance-Prod-SM-BudgetActuals` | Weekly (Monday) | 4:00 AM ET | 30 min | Finance BI Lead |
| `Executive-Prod-SM-CompanyKPIs` | Daily | 6:30 AM ET (after FactSales) | 30 min | BI CoE |

### 6.2 Refresh Dependency Chain

Some datasets depend on others. The refresh order must respect these dependencies:

```
5:00 AM ─── DimDate, DimGeography (independent, low frequency)
  │
5:30 AM ─── DimProduct, DimCustomer (independent dimensions)
  │
6:00 AM ─── FactSales (depends on dimension tables being current)
  │
6:30 AM ─── CompanyKPIs (depends on FactSales)
  │
7:00 AM ─── All dashboards available with fresh data for business hours
```

### 6.3 Refresh Failure Protocol

| Scenario | Automatic Action | Manual Action Required |
|----------|-----------------|----------------------|
| First failure | Retry after 15 minutes (automatic) | None (wait for retry) |
| Second consecutive failure | Alert sent to dataset owner via email and Teams | Owner investigates within 1 hour |
| Third consecutive failure | Alert escalated to BI CoE lead | Root cause analysis required within 4 hours |
| Failure during business hours | Stale data banner displayed on affected reports | Owner posts status update in BI Support Teams channel |

### 6.4 Refresh Monitoring

- **Microsoft Fabric monitoring hub** is checked daily by the BI CoE at 7:30 AM to confirm all refreshes succeeded.
- A weekly **Refresh Health Report** is generated showing success rates, durations, and trends. Target: 99% refresh success rate per month.
- Any dataset with refresh duration increasing by more than 50% over a 2-week period is flagged for performance investigation.

---

## 7. Roles & Responsibilities

| Role | Responsibilities |
|------|-----------------|
| **BI CoE Lead** | Owns this framework. Approves certifications and high-risk deployments. Manages deployment pipelines. Escalation point for refresh failures. |
| **BI Developer** | Builds and maintains semantic models, reports, and pipelines. Follows naming conventions and change management. Commits all changes to Git. |
| **Data Steward** | Approves access requests for their domain. Validates data quality. Participates in UAT for their business area. |
| **Business User (Viewer)** | Consumes reports and dashboards. Submits access requests. Reports data issues through the BI Support channel. |
| **IT Admin** | Manages Azure AD security groups for RLS. Configures workspace permissions. Manages Fabric capacity and gateway settings. |

---

## 8. Compliance & Audit

- All access grants, certification decisions, and deployment approvals are logged and retained for **12 months**.
- The BI CoE conducts a **quarterly audit** of workspace membership, RLS assignments, and certification status.
- Any dataset accessing sensitive data (PII, financial, HR) must have an additional **data classification label** applied and may require additional security review.

---

## Appendix A: Quick-Reference Decision Tree

```
I want to create new BI content
        │
        ├── Is it exploratory / personal? ──► Build in your personal workspace
        │                                      No naming convention required
        │
        └── Will others use it? ──► Build in DEPT-Dev workspace
                │                    Follow naming conventions
                │
                ├── Is it ready for others to test?
                │       └── Yes ──► Deploy to Test, request UAT
                │
                └── Is it approved for production?
                        └── Yes ──► Apply for Promoted or Certified badge
                                    Deploy to Prod via pipeline
```

---

*This framework is a living document. Suggest improvements by contacting the BI Center of Excellence.*