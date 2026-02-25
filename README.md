# SalesPOC APIM Deployment Repo

## Main project

- SalesPOC UI: https://github.com/csdmichael/SalesPOC.UI

This repo deploys and configures the following Azure resources in an idempotent way:

- API Management: `/subscriptions/86b37969-9445-49cf-b03f-d8866235171c/resourceGroups/ai-myaacoub/providers/Microsoft.ApiManagement/service/apim-poc-my`

## What it configures

1. **APIM API**
   - Ensures an API named `SalesPOC-API` exists in APIM.
   - Imports from App Service OpenAPI endpoint only if missing.
   - Auto-detects OpenAPI from common App Service paths:
     - `/openapi/v1.json`
     - `/openapi.json`
     - `/swagger/v1/swagger.json`
     - `/swagger.json`
     - `/v3/api-docs`
   - You can override detection by passing `-OpenApiUrl` to `scripts/deploy.ps1`.

2. **APIM MCP Server**
   - Attempts to create MCP server from `SalesPOC-API`.
   - Exposes **GET operations only** as MCP tools.
   - Applies MCP guardrail policy only when MCP server is newly created:
     - request size cap (token-budget proxy)
     - prompt-injection pattern checks
     - harmful/hate pattern checks
     - rate limit + quota


## Non-overwrite behavior

This repo is designed to **not overwrite existing resources**:

- If a target resource/config already exists, the script logs and skips it.
- Existing APIM resources are not replaced.

## GitHub Actions workflow

Workflow file: `.github/workflows/deploy-to-azure.yml`

### Required GitHub secrets

Set the following repository secrets for `azure/login@v2`:

- `AZURE_CLIENT_ID`
- `AZURE_TENANT_ID`
- `AZURE_SUBSCRIPTION_ID`

These values should come from the service principal (app registration) used by GitHub Actions.

## Run locally

```powershell
pwsh ./scripts/deploy.ps1
```

Optional: pass parameters to override defaults.

## Notes

- APIM MCP control-plane endpoints can vary by API version/release channel. The script tries multiple known preview endpoints.
- If MCP creation cannot be resolved via automation in your tenant/region, the script prints a manual fallback.
