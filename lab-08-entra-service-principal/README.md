# Lab 08 — NHI Vault App (Entra App Registration + Service Principal Login)

## Goal

Register a single-tenant Entra application, create a client secret, and authenticate via the Azure CLI as its service principal.

## Concepts

| Object | Where | What it is |
|---|---|---|
| **App registration** (application object) | Entra → App registrations | The *blueprint*: global definition of the app, lives in the home tenant |
| **Service principal** (enterprise application) | Entra → Enterprise applications | The *instance* of that blueprint in a tenant — the identity that actually signs in and gets roles |
| **Client secret** | App registration → Certificates & secrets | A password for the non-human identity (NHI) |

Creating the app registration in the portal automatically creates the matching service principal (same Application (client) ID, different Object ID).

## Steps performed

1. Entra admin center → Applications → App registrations → **New registration**
   - Name: `NHI-Vault-App`
   - Supported account types: *Accounts in this organizational directory only* (single tenant)
   - Redirect URI: left empty
2. Recorded **Application (client) ID** and **Directory (tenant) ID** from the Overview page.
3. Certificates & secrets → **New client secret** → short expiry → copied the **Value** (not the Secret ID) immediately.
4. Enterprise applications → set *Application type* filter to **All applications** → found `NHI-Vault-App` (Application ID matches the registration).
5. Logged in as the service principal from PowerShell (secret read into a variable so it never lands in shell history or the screenshot):

```powershell
$appId    = "<CLIENT_ID>"
$tenantId = "<TENANT_ID>"
$secret   = Read-Host "Client secret"

az login --service-principal -u $appId -p $secret --tenant $tenantId --allow-no-subscriptions
az account show
```

`--allow-no-subscriptions` is needed because the new service principal has no RBAC role on any subscription — without it, `az login` fails with *"No subscriptions found"*. This keeps the identity at zero Azure resource permissions (least privilege).

6. `az account show` confirms `"user": { "type": "servicePrincipal", "name": "<CLIENT_ID>" }`.

## Cleanup

```powershell
az logout
Remove-Variable secret
az login            # back to my own account
```

Then delete the client secret (or the whole app registration) in the portal.

## Why it matters (security)

- Service principals are non-human identities: no MFA, so the secret *is* the identity. Short expiry + least privilege limit the blast radius if it leaks.
- Prefer certificates, managed identities, or workload identity federation over client secrets in real workloads.
- Sign-ins show up under Entra → Sign-in logs → **Service principal sign-ins** — useful for SOC detection of abused app credentials.

## Evidence

`screenshots/sp-login-account-show.png` — `az login --service-principal … --allow-no-subscriptions` followed by `az account show`, both showing `"type": "servicePrincipal"` and `"name": "N/A(tenant level account)"`. The secret was passed as `$secret` (read with `Read-Host`), so it never appears on screen; client, tenant and subscription IDs are redacted.
