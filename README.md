Here’s a **professional, visually appealing Confluence dashboard layout** you can use to track the 8 networking Terraform modules **with descriptions, links, and live status indicators** similar to TFE governance insights (like test pass/fail counts, branch count, last run, etc.).

You can paste this straight into a Confluence page and customize further.

---

## 🌐 Cloud Networking Terraform Modules Dashboard

| Module Name                                              | Description                                | Repository / Docs | Latest TFE Status                       | Tests              | Branches        |
| -------------------------------------------------------- | ------------------------------------------ | ----------------- | --------------------------------------- | ------------------ | --------------- |
| **terraform-google-wf-cloud-networking-factory**         | Core network setup (VPC, subnetworks, IAM) | [Repo Link](#)    | 🟦 *Last Applied:* — <br> **Status:** — | TFT: — <br> BDD: — | **Branches:** — |
| **terraform-google-wf-load-balancing-factory**           | LB provisioning (HTTP(S), SSL, Internal)   | [Repo Link](#)    | 🟦 *Last Applied:* — <br> **Status:** — | TFT: — <br> BDD: — | **Branches:** — |
| **terraform-google-wf-cloud-dns-zone-factory**           | DNS zone creation & management             | [Repo Link](#)    | 🟦 *Last Applied:* — <br> **Status:** — | TFT: — <br> BDD: — | **Branches:** — |
| **terraform-google-wf-cloud-dns-factory**                | DNS records & policy automation            | [Repo Link](#)    | 🟦 *Last Applied:* — <br> **Status:** — | TFT: — <br> BDD: — | **Branches:** — |
| **terraform-google-wf-private-service-connect-factory**  | Private Service Connect endpoints          | [Repo Link](#)    | 🟦 *Last Applied:* — <br> **Status:** — | TFT: — <br> BDD: — | **Branches:** — |
| **terraform-google-f-private-regional-endpoint-factory** | Regional PSC endpoints service             | [Repo Link](#)    | 🟦 *Last Applied:* — <br> **Status:** — | TFT: — <br> BDD: — | **Branches:** — |
| **terraform-google-wf-service-directory-tactory**        | Service Directory resource automation      | [Repo Link](#)    | 🟦 *Last Applied:* — <br> **Status:** — | TFT: — <br> BDD: — | **Branches:** — |
| **terraform-google-wf-paloalto-fw-deployment-factory**   | Palo Alto FW deployment orchestration      | [Repo Link](#)    | 🟦 *Last Applied:* — <br> **Status:** — | TFT: — <br> BDD: — | **Branches:** — |

---

## 📊 Status Indicator Legend

| Icon | Meaning                     |
| ---- | --------------------------- |
| 🟩   | Success (All checks passed) |
| 🟨   | Warnings/Partial Failures   |
| 🟥   | Failure (Needs attention)   |
| 🟦   | No runs or data yet         |

---

## 📌 How To Populate the Status Columns

You can automate these cells using Confluence integrations or macros:

### **1. Terraform Cloud / Enterprise API (REST)**

Pull:

* **Last run status**
* **Latest run results**
* **Branch counts**
* **Outputs**
  Then insert using Confluence `{JSON}` or `{REST}` macros.

Example API call (replace placeholders):

```
GET https://app.terraform.io/api/v2/runs?filter[workspace]=<WORKSPACE_ID>
```

---

### **2. Test Summary (TFT & BDD)**

If you store test results in a CI system (GitHub Actions, GitLab, CircleCI):

| CI Result           | Confluence Macro |
| ------------------- | ---------------- |
| Pass/Fail           | `{CI Status}`    |
| BDD Gherkin results | `{JUnit Report}` |

Example:

```
{jira:GH-12345}{build-status}
```

---

### **3. Branch Count (from Git provider)**

Use REST macros with GitHub/GitLab APIs:

```
GET https://api.github.com/repos/<org>/<repo>/branches
```

Show count or list.

---

## 📌 Example Confluence Macros to Embed

### **REST Call Macro**

```
{rest}
url=https://api.terraform.io/api/v2/runs?filter[workspace]=<ID>
title=Terraform Last Run
render=table
{rest}
```

---

## 🎯 Optional Enhancements

### 🔹 Filters & Toggles

Add filters so users can show:

* Failing modules only
* Tests with errors
* Branches > X

### 🔹 Visual Charts

Use `{chart}` macros to:

* Show success rate trends
* Test pass/fail over time
* Branch count changes

### 🔹 Alert Sections

Add automatic highlights for:

* Failed Terraform runs
* Stale modules > 7 days without tests
* High branch count

---

## 📥 Quick Copy Template (Without Status Data)

```markdown
|| Module || Description || Repo/Docs || Latest TFE Status || Tests || Branches ||
| **terraform-google-wf-cloud-networking-factory** | Core network setup | [Link](#) | 🟦 | | |
| **terraform-google-wf-load-balancing-factory** | Load balancing | [Link](#) | 🟦 | | |
| **terraform-google-wf-cloud-dns-zone-factory** | DNS zone automation | [Link](#) | 🟦 | | |
| **terraform-google-wf-cloud-dns-factory** | DNS records | [Link](#) | 🟦 | | |
| **terraform-google-wf-private-service-connect-factory** | Private SC | [Link](#) | 🟦 | | |
| **terraform-google-f-private-regional-endpoint-factory** | Regional endpoints | [Link](#) | 🟦 | | |
| **terraform-google-wf-service-directory-tactory** | Service Directory | [Link](#) | 🟦 | | |
| **terraform-google-wf-paloalto-fw-deployment-factory** | Palo Alto FW | [Link](#) | 🟦 | | |
```

---

## 🎤 Want This As A Live Template?

If you tell me:
✔ The TFE workspace IDs,
✔ Git repo links,
✔ CI system in use,
✔ Test result formats,

I’ll generate a **filled-out Confluence page with embedded macros** and working status updates.
