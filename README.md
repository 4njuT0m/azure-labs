# Azure Labs

A collection of Microsoft Azure exercises and infrastructure templates.

## Labs

| # | Lab | Topic | Deployed with | Status |
|---|-----|-------|---------------|--------|
| 01 | [Internal Load Balancer](./lab-01-internal-load-balancer/) | Deploy and test an internal load balancer | ARM template | ✅ Complete |
| 02 | [Application Gateway](./lab-02-application-gateway/) | Deploy and test an L7 gateway across two backend VMs | Azure CLI | ✅ Complete |

## Structure

Each lab folder contains:

- `README.md` — task, architecture, and notes
- `screenshots/` — screenshots of the completed task

ARM labs also contain:

- `template.json` — cleaned ARM template (no secrets)
- `parameters.example.json` — copy this, fill in password, never commit

CLI labs contain instead:

- `lab<NN>-artifacts/` — evidence exported from the live deployment: `template.json` (exported ARM), full resource JSON (e.g. `appgw.json`), `resources.txt`, and command output that proves the lab worked (`backend-health.txt`, `curl-alternating.txt`, `nsg-rules.txt`, …)

## How to Redeploy Any Lab

**ARM labs**

1. Copy `parameters.example.json` → `parameters.json`
2. Fill in your `adminPassword`
3. In Azure Portal → search **Deploy a custom template** → upload `template.json`

> `parameters.json` is in `.gitignore` — never commit it.

**CLI labs**

Follow the Microsoft Quickstart linked from the lab's `README.md`, substituting your own admin password / SSH key. The exported `template.json` under `lab<NN>-artifacts/` is a reference of the resulting resource group, not the deployment mechanism.

> Delete the resource group when finished — Application Gateway bills continuously.
