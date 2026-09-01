\# Azure Labs



A collection of Microsoft Azure exercises and infrastructure templates.



\## Labs



| # | Lab | Topic | Status |

|---|-----|-------|--------|

| 01 | \[Internal Load Balancer](./lab-01-internal-load-balancer/) | Deploy and test an internal load balancer | ✅ Complete |



\## Structure

Each lab folder contains:

\- `README.md` — task, architecture, and notes

\- `template.json` — cleaned ARM template (no secrets)

\- `parameters.example.json` — copy this, fill in password, never commit

\- Screenshot of completed task



\## How to Redeploy Any Lab

1\. Copy `parameters.example.json` → `parameters.json`

2\. Fill in your `adminPassword`

3\. In Azure Portal → search \*\*Deploy a custom template\*\* → upload `template.json`

4\. Or via CLI: `az deployment group create --template-file template.json --parameters parameters.json`



> `parameters.json` is in `.gitignore` — never commit it.

