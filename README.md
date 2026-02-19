# KioskApp Requirements

Use this checklist to configure, package, and deploy the app to kiosk PCs.

## 1) What's contained
- Source code and solution.
- Publish outputs:
  - `win-x64` package

- Excluded:
  - `win-arm64` package (Build/Publish)

## 2) Build/Publish prerequisites (only if building from source)
- Windows build machine.
- .NET SDK 10.x installed.
- Access to NuGet feed(s), including `https://api.nuget.org/v3/index.json` (or approved internal mirror).
- Internet/network access to Azure AD and Power BI endpoints.

## 3) Required configuration
Update `KioskApp/appsettings.json` values:

- `AzureAd.ClientId`
- `AzureAd.TenantId`
- `AzureAd.ClientSecret`
- `PowerBI.WorkspaceId`
- `PowerBI.ReportId`

Security note:
- Do not store production secrets in source control.
- Prefer secret injection at deployment time (environment variables, secret store, or Key Vault).

## 4) Power BI / Entra ID prerequisites
- Service principal app registration exists in Microsoft Entra ID.
- Service principal has required Power BI API permissions with admin consent.
- Service principal is added to the target Power BI workspace with sufficient access.
- Target report exists and is published in the specified workspace.

## 5) Kiosk PC prerequisites (runtime)
If deploying **self-contained** publish output:

- .NET runtime is **not required** on kiosk PCs.
- CPU architecture must match package:
  - x64 kiosks -> deploy `win-x64`
  - ARM64 kiosks -> deploy `win-arm64`

Also required:
- Supported browser engine available on device (Microsoft Edge/Chrome/Firefox).
- Outbound HTTPS allowed to Microsoft identity + Power BI services.
- Local security policy allows app execution from deployment folder.

## 6) Network and firewall requirements
Allow outbound HTTPS (443) to required Microsoft endpoints, including:
- `login.microsoftonline.com`
- `api.powerbi.com`
- Power BI embed/content domains used by your tenant

If your environment uses SSL inspection/proxy controls, verify tokens and embed content are not blocked.

## 7) Build and publish commands (from source)
From repository root:

- Restore/build:
  - `dotnet restore KioskApp/KioskApp.csproj`
  - `dotnet build KioskApp/KioskApp.csproj -c Release`

- Publish x64 (self-contained single file):
  - `dotnet publish KioskApp/KioskApp.csproj -c Release -r win-x64 --self-contained true /p:PublishSingleFile=true /p:PublishTrimmed=false`

- Publish ARM64 (self-contained single file):
  - `dotnet publish KioskApp/KioskApp.csproj -c Release -r win-arm64 --self-contained true /p:PublishSingleFile=true /p:PublishTrimmed=false`

Output folders:
- `KioskApp/bin/Release/net10.0/win-x64/publish`
- `KioskApp/bin/Release/net10.0/win-arm64/publish`

## 8) Deployment checklist for kiosk devices

- Copy the correct published folder contents to target device.
- Configure startup/launcher mechanism (service, shell replacement, scheduled task, or kiosk launcher).
- Validate app can reach identity and Power BI endpoints.
- Validate report loads with expected access.
- Reboot test to confirm unattended startup behavior.

## 9) Smoke test after deployment
- App starts without interactive auth prompts.
- Target report renders successfully.
- Navigation/session remains stable for extended runtime.
- Device restart recovers application automatically.

## 10) Common issues

- **401/403 from Power BI**: service principal lacks workspace/API permissions or wrong tenant settings.
- **Report not found**: incorrect `WorkspaceId`/`ReportId`.
- **Token acquisition fails**: invalid `ClientId`/`TenantId`/`ClientSecret` or blocked outbound network.
- **App starts but blank/embed fails**: proxy/firewall/certificate inspection blocking embed resources.
