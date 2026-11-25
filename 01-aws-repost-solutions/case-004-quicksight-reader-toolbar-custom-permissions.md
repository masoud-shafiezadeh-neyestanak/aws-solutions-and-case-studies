# Case 004 — QuickSight Reader Toolbar Visibility and Custom Permissions

> An independent technical research note based on an AWS re:Post discussion. The scenario and analysis are summarized in original wording; the original question and community answer are not reproduced.

## 1. Source Discussion

- **Platform:** AWS re:Post
- **Discussion:** [Hiding Reader toolbar buttons (View Data, Scheduling, View alerts)](https://repost.aws/questions/QUS8PA0Y3FSG6I2fRODnIGog/hiding-reader-toolbar-buttons-view-data-scheduling-view-alerts)
- **Date reviewed:** 2026-11-25
- **Product terminology:** Amazon Quick / Amazon QuickSight (the documentation and UI terminology may vary during the product's naming transition)
- **My participation:** Answer published in the discussion
- **Thread outcome:** Answer accepted by the question author, received one community vote, and was marked Expert Approved on re:Post

These are community-platform outcomes. They are not an official AWS endorsement, certification, or guarantee that the proposed checks resolve every environment.

## 2. Technical Scenario

An administrator wanted to simplify the dashboard experience for Reader users while preserving CSV, Excel, and PDF exports.

The Reader population was assigned a Custom Permissions profile that denied email-report creation and subscription capabilities, as well as threshold-alert creation and update capability. Despite those restrictions, dashboard viewers continued to see the **View Data** entry and, depending on the observed session, either **Scheduling** or **View alerts**. The corresponding panels appeared to be view-only.

The administrator had also changed dashboard publishing options related to filtering, visual menus, executive summaries, and data stories, but those changes did not remove the View Data entry.

The questions were whether the three toolbar entries could be hidden independently, whether their visibility should vary when permissions and dashboard settings appear unchanged, and how to retain interactive UI export options.

## 3. AWS Services and Concepts

- **Product:** Amazon Quick / Amazon QuickSight
- **User role:** Reader
- **Authorization model:** Custom Permissions profiles
- **Relevant capabilities:**
  - `CreateAndUpdateDashboardEmailReports`
  - `SubscribeDashboardEmailReports`
  - `CreateAndUpdateThresholdAlerts`
  - `ExportToCsv`
  - `ExportToExcel`
  - `ExportToPdf`
- **Dashboard features:** Reader-generated reports, pixel-perfect report sheets, threshold alerts, dashboard toolbar and visual menus

## 4. Key Technical Questions

1. Does the documented Custom Permissions capability model expose separate controls for hiding `View Data`, `Scheduling`, and `View alerts`?
2. Do denying email-report and threshold-alert actions necessarily hide the corresponding dashboard navigation entries?
3. Can CSV, Excel, and PDF exports remain available while email-report and alert actions are restricted?
4. What dashboard or account prerequisites affect Reader-generated report scheduling?
5. How should administrators investigate toolbar entries that appear to vary between sessions when the effective permissions are believed to be unchanged?

## 5. Investigation and Evidence

The current AWS documentation establishes the following:

| Finding | Evidence | Confidence |
|---|---|---|
| `CreateAndUpdateDashboardEmailReports` controls the ability to create and update email reports. | Amazon Quick API — Capabilities | Documented |
| `SubscribeDashboardEmailReports` controls the ability to subscribe to email reports. | Amazon Quick API — Capabilities | Documented |
| `CreateAndUpdateThresholdAlerts` controls creating and updating threshold alerts. | Amazon Quick API — Capabilities | Documented |
| `ExportToCsv`, `ExportToExcel`, and `ExportToPdf` independently control UI exports to those formats. | Amazon Quick API — Capabilities | Documented |
| Scheduled-report exports have separate capabilities: `ExportToCsvInScheduledReports`, `ExportToExcelInScheduledReports`, and `ExportToPdfInScheduledReports`. | Amazon Quick API — Capabilities | Documented |
| The documented Capabilities schema does not list separate fields named `ViewData`, `Scheduling`, or `ViewAlerts`. | Amazon Quick API — Capabilities | Documented for the current schema |
| Reader-generated reports are a supported workflow for dashboard viewers when an author has configured a prompted pixel-perfect report. | User Guide — Creating a reader generated report | Documented |
| The Add control for creating a reader-generated report is unavailable when the dashboard has no pixel-perfect sheet or the Quick account lacks the Pixel perfect reports add-on. | User Guide — Creating a reader generated report | Documented |
| Threshold alerts can be viewed and managed through the Alerts area; documented management tasks include editing, viewing history, turning alerts on or off, and deleting them. | User Guide — Managing Threshold Alerts | Documented |
| Dashboard publishing options can disable the visual menu and visual download options, as well as selected data-point interactions. | User Guide — Publishing dashboards | Documented |
| Custom Permissions profiles can be applied at account, role, and user levels. The most specific level wins: user overrides role, and role overrides account. | User Guide — Security in Quick | Documented |

### Evidence Boundary

The current public API schema is a list of supported Custom Permissions capability fields. It does not expose exact fields named `ViewData`, `Scheduling`, or `ViewAlerts`. This supports the conclusion that there is no separately documented capability with those exact names in the current schema. It does not prove that no other product configuration, account feature, or future UI option could affect those entries.

The documentation explains how Readers can use scheduling and how alert management works, but it does not provide a definitive mapping from every toolbar entry's visibility to every Custom Permissions capability. The documentation reviewed also does not describe day-to-day random changes in toolbar visibility when the effective permissions and dashboard configuration remain unchanged.

## 6. Analysis

The issue involves two different layers:

- **Action authorization:** Whether a Reader can create, update, subscribe to, export, or manage a particular resource.
- **Navigation visibility:** Whether the dashboard UI displays an entry that opens a panel or workflow.

Denying an action does not necessarily mean that the associated navigation item is hidden. A view-only panel may remain visible even when some actions within that panel are unavailable.

### 6.1 View Data

The documented dashboard experience includes a dashboard menu bar with options for working with data. The publishing options separately control the visual menu, visual download options, and selected data-point interactions.

The documented Custom Permissions capability schema does not contain a dedicated `ViewData` field. Disabling the visual menu or data-point options should therefore not be assumed to remove a distinct dashboard-level Data entry.

The broader `Dashboard` capability controls dashboard-related actions. Denying it merely to remove one toolbar entry could affect much more than the single UI element the administrator is trying to hide.

### 6.2 Scheduling

Amazon Quick supports Reader-generated reports. A Reader can use a prompted report when the author has configured a pixel-perfect report. The Reader-generated report workflow requires the relevant pixel-perfect sheet and the Pixel perfect reports add-on; without either prerequisite, the Add control is not available.

This explains why the Scheduling experience can depend on dashboard content and account feature availability, not only on the Reader's Custom Permissions profile. The public documentation reviewed does not define a separate toolbar-visibility switch for Scheduling.

### 6.3 View alerts

The capability `CreateAndUpdateThresholdAlerts` is documented as controlling the creation and updating of threshold alerts. AWS separately documents an Alerts area for viewing and managing existing alerts and their history.

No separate `ViewAlerts` field appears in the documented Custom Permissions capability schema. The absence of such a field means administrators should not assume that denying alert creation and update will hide the entire Alerts entry.

### 6.4 Preserving UI exports

The following capabilities are separate fields in the Custom Permissions schema:

- `ExportToCsv`
- `ExportToExcel`
- `ExportToPdf`

They can be allowed while the email-report and threshold-alert capabilities are denied. Scheduled-report export permissions are separate fields and should be reviewed independently if scheduled reports are in scope.

### 6.5 Inconsistent visibility between days

AWS documents Custom Permissions precedence across three levels:

1. User-level profile
2. Role-level profile
3. Account-level profile

The most specific profile takes precedence. An administrator should verify the effective assignment at all three levels rather than checking only the profile visible in one part of the console.

Other relevant variables to compare include the same Reader identity, the same dashboard and published version, whether a pixel-perfect sheet is present, add-on availability, and the AWS Region in which the dashboard is being viewed.

The available documentation does not establish that toolbar entries should randomly appear and disappear from one day to the next under otherwise identical conditions. If the behavior remains inconsistent after these checks, it should be treated as an issue requiring reproducible evidence rather than as documented expected behavior.

## 7. Recommended Troubleshooting Approach

1. **Inspect the effective Custom Permissions profile.** Check account-, role-, and user-level assignments. Apply the documented precedence rule: user overrides role, and role overrides account.
2. **Confirm the intended action restrictions.** Verify the values for `CreateAndUpdateDashboardEmailReports`, `SubscribeDashboardEmailReports`, and `CreateAndUpdateThresholdAlerts`.
3. **Keep the UI export capabilities separate.** Confirm that `ExportToCsv`, `ExportToExcel`, and `ExportToPdf` remain allowed. Review the `...InScheduledReports` variants separately if scheduled reports are also in scope.
4. **Inspect dashboard publishing settings.** Check whether the visual menu, visual download options, and data-point interactions are enabled. Do not treat these options as controls for every dashboard-level toolbar entry.
5. **Check Scheduling prerequisites.** Confirm whether the dashboard includes a pixel-perfect report sheet and whether the account has the Pixel perfect reports add-on. Review what the Reader can actually do inside the Scheduling pane.
6. **Check alert context.** Determine whether alerts exist for the Reader and dashboard, and distinguish viewing alert history from creating, editing, enabling, disabling, or deleting alerts.
7. **Reproduce the visibility difference consistently.** Use the same Reader, same dashboard, same Region, same published dashboard version, and comparable browser/session conditions. Record timestamps and screenshots of the full toolbar and the panel opened by each entry.
8. **Escalate with evidence.** If the UI still changes unexpectedly while the effective permissions and dashboard configuration are unchanged, contact AWS Support. Provide the timestamps, dashboard and user context, relevant permission assignments, Region, screenshots, and a concise reproduction sequence.

## 8. Validation Status

- [x] Current Custom Permissions capability schema reviewed
- [x] UI export capabilities distinguished from scheduled-report export capabilities
- [x] Reader-generated report documentation reviewed
- [x] Threshold-alert permissions and management documentation reviewed
- [x] Custom Permissions precedence across user, role, and account levels confirmed
- [ ] Toolbar visibility reproduced in an independent Quick environment
- [ ] Day-to-day visibility inconsistency independently reproduced
- [ ] AWS Support confirmed whether the changing UI visibility is a product issue
- [x] Answer posted in the linked re:Post discussion
- [x] Answer accepted by the question author
- [x] Answer received one community vote
- [x] Answer marked Expert Approved on re:Post

**Conclusion:** The documented Custom Permissions API controls actions such as report creation, report subscription, alert creation/update, and UI exports. It does not expose separately named capabilities for hiding the three toolbar entries discussed. Scheduling and Alerts have their own documented workflows and prerequisites. The observed day-to-day visibility change is not explained by the public documentation reviewed; a controlled comparison and, if needed, AWS Support investigation are appropriate.

## 9. What I Learned

- Permissions that control an action and settings that control whether a UI element is displayed are not necessarily the same thing.
- A navigation entry can remain visible even when the actions behind it are restricted.
- Reader-generated reports introduce dashboard-content and account-feature prerequisites beyond the basic Reader role.
- UI exports and scheduled-report exports use separate Custom Permissions fields.
- Custom Permissions can be applied at account, role, and user levels, with the most specific level taking precedence.
- When a UI issue appears inconsistent, a reproducible comparison must hold the user, dashboard, Region, published version, and session conditions constant.
- A careful answer should distinguish documented behavior from a UI explanation that still requires independent reproduction or AWS Support confirmation.

## 10. References

1. [Amazon Quick API — Capabilities](https://docs.aws.amazon.com/quicksight/latest/APIReference/API_Capabilities.html) — Definitions of report, subscription, threshold-alert, dashboard, and export capabilities.
2. [Amazon Quick — Security in Quick](https://docs.aws.amazon.com/quick/latest/userguide/best-practices-security.html) — Least privilege and Custom Permissions precedence across user, role, and account levels.
3. [Creating a reader generated report](https://docs.aws.amazon.com/quick/latest/userguide/reader-scheduling.html) — Reader-generated reports and pixel-perfect report/add-on requirements.
4. [Alert Permissions](https://docs.aws.amazon.com/quick/latest/userguide/threshold-alerts-permissions.html) — Custom permissions for controlling who can set threshold alerts.
5. [Managing Threshold Alerts](https://docs.aws.amazon.com/quick/latest/userguide/threshold-alerts-managing.html) — Editing, viewing history, enabling/disabling, and deleting alerts.
6. [Publishing dashboards](https://docs.aws.amazon.com/quick/latest/userguide/creating-a-dashboard.html) — Dashboard publishing options for visual menus, downloads, and data-point interactions.
7. [Interacting with Amazon Quick Sight dashboards](https://docs.aws.amazon.com/quick/latest/userguide/exploring-dashboards.html) — Dashboard menu bar and viewer interaction options.
8. [UpdateUserCustomPermission](https://docs.aws.amazon.com/quicksight/latest/APIReference/API_UpdateUserCustomPermission.html) — Applying a profile at user level.
9. [UpdateRoleCustomPermission](https://docs.aws.amazon.com/quicksight/latest/APIReference/API_UpdateRoleCustomPermission.html) — Applying a profile at role level.
10. [UpdateAccountCustomPermission](https://docs.aws.amazon.com/quicksight/latest/APIReference/API_UpdateAccountCustomPermission.html) — Applying a profile at account level.
11. [Original AWS re:Post discussion](https://repost.aws/questions/QUS8PA0Y3FSG6I2fRODnIGog/hiding-reader-toolbar-buttons-view-data-scheduling-view-alerts) — Community question and answer status.

## 11. Privacy and Attribution Review

- [x] Scenario summarized in original wording
- [x] Other users' question and answers not reproduced
- [x] No account-specific or personal information included
- [x] Official AWS documentation linked
- [x] Accepted, vote, and Expert Approved statuses attributed to re:Post
- [x] No claim that the toolbar behavior was independently reproduced
- [x] No unsupported claim that AWS guarantees random or stable toolbar visibility

---

This is an independent learning and research note. It is not official AWS guidance or AWS Support confirmation.
