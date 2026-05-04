<div align="center">

<img src="logo.png" alt="Mazhai DeployFusion" width="120"/>

# Mazhai DeployFusion

**Enterprise Application Packaging & Deployment Automation**

[![Version](https://img.shields.io/badge/version-1.0.0-6366F1?style=flat-square)](https://github.com/mazhaistudios/DeployFusion/releases)
[![Platform](https://img.shields.io/badge/platform-Windows-blue?style=flat-square)](https://github.com/mazhaistudios/DeployFusion)
[![.NET](https://img.shields.io/badge/.NET-8.0-512BD4?style=flat-square)](https://dotnet.microsoft.com/en-us/download/dotnet/8.0)
[![License](https://img.shields.io/badge/license-MIT-green?style=flat-square)](LICENSE)

*Package once. Deploy everywhere. No scripts required.*

</div>

---

## What is Mazhai DeployFusion?

Mazhai DeployFusion is a **free, open-source Windows desktop tool** built for SCCM (Microsoft Configuration Manager) and Intune administrators. It automates the entire application packaging and deployment pipeline — from a raw `.exe` or `.msi` installer to a fully deployed, collection-targeted SCCM application or Intune Win32 app — in a single click.

Tasks that previously required 30–60 minutes of manual console work (creating the app, writing PSADT scripts, creating collections, setting up deployments) are completed in under **2 minutes**, with zero scripting knowledge required.

---

## ✨ Features at a Glance

| Feature | Description |
|---|---|
| 🚀 **One-Click SCCM Deployment** | Creates the app, PSADT wrapper, deployment types, 3 collections, and 3 deployments automatically |
| 📦 **Intune Win32 Publishing** | Packages `.intunewin`, uploads via MS Graph API, creates Azure AD groups and assignments |
| 🎨 **Auto Icon Fetching** | Fetches high-quality app icons automatically from the web |
| 📋 **PSADT v4 Wrapper Generation** | Generates a production-ready PSADT v4 deployment script from your installer |
| 🗂 **Manage Apps** | Browse, edit, rename, retire, redistribute, and delete SCCM applications from a single view |
| 📊 **Health Dashboard** | Live compliance statistics across all deployed SCCM applications |
| 🔐 **Dual Auth Modes** | Supports interactive (user login) and unattended (Entra App Registration / client secret) for Intune |
| 🌍 **Multi-Profile** | Switch between DEV, UAT, PROD SCCM environments instantly |
| 🔒 **Encrypted Secrets** | Client secrets are encrypted using Machine-Level DPAPI — safe for shared admin servers |
| 📝 **Audit Logging** | Every operation is logged to a local SQLite database with operator name, machine, and status |
| 🖱 **Drag & Drop** | Drop an installer file directly onto the window to load it |

---

## 📸 Screenshots

> *Screenshots of the Package App, Manage Apps, Dashboard, and Settings screens.*

*(Coming soon — see the [wiki](https://github.com/mazhaistudios/DeployFusion/wiki) for a full walkthrough.)*

---

## 🔧 Prerequisites

Before installing or building DeployFusion, ensure the following are in place:

### On the machine running DeployFusion

| Requirement | Details |
|---|---|
| **Windows 10/11 or Windows Server 2019+** | x64 only |
| **.NET 8.0 Desktop Runtime** | [Download here](https://dotnet.microsoft.com/en-us/download/dotnet/8.0) |
| **SCCM Admin Console** | Must be installed. The tool auto-detects it at `E:\SCCM\AdminConsole\bin`, `C:\Program Files (x86)\Microsoft Configuration Manager\AdminConsole\bin`, or `C:\Program Files (x86)\Microsoft Endpoint Manager\AdminConsole\bin` |
| **Network access to SCCM Site Server** | TCP port 135 (RPC) must be open |
| **SCCM Administrator rights** | The account running the tool needs Full Administrator role in SCCM |

### For Intune publishing (optional)

| Requirement | Details |
|---|---|
| **Microsoft Entra App Registration** | Required for unattended mode (see [Entra Setup](#-entra-app-registration-setup)) |
| **Internet connectivity** | Required to reach `graph.microsoft.com` |
| **IntuneWinAppUtil.exe** | Downloaded automatically on first Intune publish |

---

## 🚀 Installation

### Option 1: Install from MSI (Recommended)

1. Download the latest `MazhaiDeployFusion-1.0.0.msi` from the [Releases](https://github.com/mazhaistudios/DeployFusion/releases) page.
2. Run the installer. It will install to `C:\Program Files\Mazhai DeployFusion\` by default.
3. Launch **Mazhai DeployFusion** from the Start Menu.

### Option 2: Build from Source

```powershell
# 1. Clone the repository
git clone https://github.com/mazhaistudios/DeployFusion.git
cd deployfusion

# 2. Restore NuGet packages
dotnet restore SCCMAutomationTool.csproj

# 3. Build (auto-detects SCCM Admin Console path)
dotnet build SCCMAutomationTool.csproj --configuration Release

# 4. Run
.\bin\Release\net8.0-windows\SCCMAutomationTool.exe
```

> **Note:** Building requires the SCCM Admin Console to be installed on the build machine, as the SCCM SDK DLLs are referenced from the console's `bin` folder.

---

## ⚙️ First-Time Setup

### 1. Configure Your SCCM Environment

1. Open **Settings** from the left navigation bar.
2. Fill in the following fields:

| Field | Description | Example |
|---|---|---|
| **Profile Name** | A friendly name for this environment | `PROD` |
| **SCCM Server** | Hostname of your SMS Provider / Site Server | `SCCM01.contoso.com` |
| **Site Code** | Your 3-character SCCM site code | `PS1` |
| **Default Shared Drive Path** | UNC path where application source folders are created | `\\sccm01\Sources\Applications` |

3. Click **🔌 Test Connection** to verify connectivity.
4. Click **💾 Save Settings**. The tool validates the SCCM connection before saving.

### 2. Configure Intune (Optional)

If you plan to publish applications to Intune:

#### Interactive Login (Simpler)

1. Leave the **"Use Unattended App Registration"** checkbox **unchecked**.
2. When you publish to Intune, a browser window will open for you to sign in with your Microsoft account. Your session is cached for future operations.

#### Unattended / Service Account Mode (Recommended for Servers)

1. Check **"Use Unattended App Registration (Client Secret instead of Interactive Login)"**.
2. Enter the three credentials from your Entra App Registration:

| Field | Where to find it |
|---|---|
| **Directory (Tenant) ID** | Entra portal → App registrations → Your app → Overview |
| **Application (Client) ID** | Entra portal → App registrations → Your app → Overview |
| **Client Secret** | Entra portal → App registrations → Your app → Certificates & secrets |

3. Click **Save Settings**. The secret is encrypted using **Machine-Level DPAPI** — it cannot be decrypted on another machine.

---

## 🔐 Entra App Registration Setup

To use unattended Intune publishing, create an App Registration in Microsoft Entra:

1. Sign in to [https://entra.microsoft.com](https://entra.microsoft.com)
2. Navigate to **App registrations** → **New registration**
3. Give it a name (e.g., `DeployFusion-Automation`) and click **Register**
4. Note the **Directory (tenant) ID** and **Application (client) ID** from the Overview page
5. Go to **Certificates & secrets** → **New client secret** → Set expiry → **Add**
6. Copy the secret **value** immediately (it won't be shown again)
7. Go to **API permissions** → **Add a permission** → **Microsoft Graph** → **Application permissions**
8. Add the following permissions:

| Permission | Purpose |
|---|---|
| `DeviceManagementApps.ReadWrite.All` | Create and manage Win32 apps in Intune |
| `Group.ReadWrite.All` | Create and manage Azure AD security groups |

9. Click **Grant admin consent**

---

## 📦 Packaging & Deploying an Application

### SCCM Deployment

1. Click **📦 Package App** in the left navigation bar.
2. Click **Browse** (or drag and drop your installer file onto the window).
3. The tool automatically pre-fills:
   - Application name (from the file name)
   - Publisher / Manufacturer (from file metadata)
   - Install arguments (from the MSI product database or common patterns)
   - Application icon (fetched automatically from the web)
4. Review and adjust the fields if needed.
5. Choose your deployment options:
   - **Destination Path** — overrides the profile default if needed
   - **Distribute to All DPs** — tick to trigger content distribution immediately
   - **Also publish to Intune** — tick to run the Intune pipeline simultaneously
6. Click **🚀 Build & Publish to SCCM**.
7. Review the confirmation dialog, then click **Deploy**.

**What happens automatically:**

```
Step 1 ── Prepare file system
         └─ Creates \\server\share\AppName\
         └─ Copies installer to Files\
         └─ Generates PSADT v4 wrapper (Invoke-AppDeployToolkit.ps1)
         └─ Fetches and places application icon in SupportFiles\

Step 2 ── Create SCCM Application
         └─ Creates SMS_Application via WMI/SDK
         └─ Adds Script Deployment Type with registry detection
         └─ Sets application icon

Step 3 ── Distribute content (optional)
         └─ Sends to "All Distribution Points" group

Step 4 ── Create 3 Device Collections (in SoftwareDistribution folder)
         └─ AppName - Required
         └─ AppName - Uninstall
         └─ AppName - Available

Step 5 ── Create 3 Deployments
         └─ Required install → Required collection
         └─ Required uninstall → Uninstall collection
         └─ Available install → Available collection
```

### Intune Deployment

When **"Also publish to Intune"** is checked (or when triggered standalone):

```
Step 1 ── Download IntuneWinAppUtil.exe (first time only, from Microsoft GitHub)
Step 2 ── Wrap installer + PSADT in .intunewin package
Step 3 ── Authenticate (interactive browser or app secret)
Step 4 ── Create Win32 app in Intune via MS Graph API
Step 5 ── Upload encrypted package via Azure Blob Block protocol
Step 6 ── Create 3 Azure AD Security Groups (Available, Required, Uninstall)
Step 7 ── Assign groups to the Intune app
```

---

## 🗂 Managing Applications

Click **Manage Apps** to browse all SCCM applications.

| Action | Description |
|---|---|
| **Refresh Apps** | Reload the application list from SCCM |
| **Export CSV** | Export the list to a CSV file |
| **Select an app** | Opens a details sidebar with full metadata, detection methods, and collection info |
| **Edit Details** | Update Description, Publisher, and Version directly from the sidebar |
| **Rename** | Renames the app, all collections, all deployments, and the source folder atomically |
| **⏸ Retire / ▶️ Restore** | Retire (disable) or restore an application |
| **Redistribute** | Re-trigger content distribution to all DPs |
| **🗑 Delete** | Removes deployments, content from DPs, collections, the SCCM application, and the source folder |

---

## 📊 Health Dashboard

The Dashboard shows a real-time compliance summary across all deployed applications, sourced live from SCCM:

- **Total Success** — devices with the app installed successfully
- **In Progress** — devices with a pending or in-progress deployment
- **Total Errors** — devices with a failed deployment

The data grid shows per-collection compliance percentages with an inline progress bar.

---

## 🏗 Architecture & Technical Details

### Technology Stack

| Layer | Technology |
|---|---|
| UI Framework | WPF (.NET 8.0-windows) |
| SCCM Connectivity | WMI / WqlConnectionManager (native SCCM SDK) |
| SCCM Application Model | `Microsoft.ConfigurationManagement.ApplicationManagement` SDK |
| Intune API | Microsoft Graph REST API (`/beta/deviceAppManagement`) |
| Authentication | MSAL.NET (`Microsoft.Identity.Client`) |
| Secret Encryption | `System.Security.Cryptography.ProtectedData` (DPAPI, Machine scope) |
| Audit Logging | SQLite via `Microsoft.Data.Sqlite` |
| PSADT Packaging | PSAppDeployToolkit v4 (embedded as `PSADT.zip`) |
| Intune Packaging | IntuneWinAppUtil.exe (auto-downloaded from Microsoft) |
| Icon Fetching | `IconFetcher.cs` — multi-source web fetching |
| Settings | JSON file at `%ProgramData%\MazhaiCloud\SCCMAutomationTool\appsettings_v2.json` |

### PSADT Wrapper Generation

For every application packaged, the tool generates a complete PSADT v4 deployment package:

- **Extracts** the bundled PSADT v4 template from `PSADT.zip` (embedded resource)
- **Configures** `Invoke-AppDeployToolkit.ps1` with your app name, vendor, version, install/uninstall commands
- **Injects** a branding registry block under `HKLM:\SOFTWARE\Mazhai\{identifier}` for detection
- **Handles** both MSI (`Start-ADTMsiProcess`) and EXE (`Start-ADTProcess`) installers automatically
- **Writes** the standard post-install confirmation prompt with the app name

### Detection Method

All applications use a standardized dual-registry detection method:

```
HKEY_LOCAL_MACHINE\SOFTWARE\Mazhai\Mazhai-{AppSlug}-{arch}-En
  Name     = {Vendor}_{identifier}_ALL_1.0-0001_EN   (String, equals)
  Revision = 0001                                      (String, equals)
```

This is consistent across both SCCM and Intune deployments.

### Configuration File

Settings are stored at:
```
%ProgramData%\MazhaiCloud\SCCMAutomationTool\appsettings_v2.json
```

```json
{
  "Profiles": [
    {
      "ProfileName": "PROD",
      "SccmServer": "SCCM01.contoso.com",
      "SiteCode": "PS1",
      "DefaultDestinationPath": "\\\\server\\Sources\\Applications"
    }
  ],
  "ActiveProfileIndex": 0,
  "IntuneTenantId": "your-tenant-id",
  "IntuneClientId": "your-client-id",
  "UseIntuneAppSecret": true,
  "IntuneClientSecretEncrypted": "<DPAPI encrypted base64>"
}
```

### Audit Log

Every deployment, deletion, and rename is persisted to a SQLite database:
```
%ProgramData%\MazhaiCloud\DeployFusion\Logs\{MachineName}\AuditLog.db
```

The database schema:

| Column | Type | Description |
|---|---|---|
| `Id` | INTEGER | Auto-incremented primary key |
| `Timestamp` | TEXT | UTC timestamp (ISO 8601) |
| `OperationType` | TEXT | `DEPLOY`, `DELETE`, `RENAME`, `RETIRE`, etc. |
| `AppName` | TEXT | Application name |
| `Operator` | TEXT | Windows username |
| `Machine` | TEXT | Machine hostname |
| `Status` | TEXT | `SUCCESS`, `FAILED`, `PENDING` |
| `Details` | TEXT | Error message or success summary |

Entries older than **30 days** are automatically purged on startup (PENDING entries are preserved).

---

## 🤝 Contributing

Contributions are welcome! Please follow these steps:

1. **Fork** the repository
2. Create a feature branch: `git checkout -b feature/your-feature-name`
3. Make your changes and ensure the project builds: `dotnet build`
4. **Commit** with a descriptive message: `git commit -m "Add: description of change"`
5. **Push** to your fork: `git push origin feature/your-feature-name`
6. Open a **Pull Request** against `main`

### Building Requirements for Contributors

- Visual Studio 2022 or JetBrains Rider
- .NET 8 SDK
- SCCM Admin Console installed locally (for SDK DLL resolution)

### Project Structure

```
SCCMAutomationTool/
├── MainWindow.xaml          # Main UI — all panels (Package, Manage, Dashboard, Settings, About)
├── MainWindow.xaml.cs       # UI code-behind and event handlers
├── SCCMAutomation.cs        # All SCCM operations (WMI/SDK — build, manage, delete, rename)
├── IntuneAutomation.cs      # Intune pipeline (package, upload, assign via MS Graph)
├── PSADTBuilder.cs          # PSADT v4 wrapper generation
├── ConfigManager.cs         # Settings load/save/encrypt
├── AuditDatabase.cs         # SQLite audit log
├── IconFetcher.cs           # Web-based icon retrieval
├── App.xaml                 # WPF application entry point and global resource dictionaries
├── App.xaml.cs              # App startup logic
├── PSADT.zip                # Embedded PSADT v4 template
├── logo.png                 # Application logo
├── App.ico                  # Application icon
└── Setup/
    └── Product.wxs          # WiX v4 MSI installer definition
```

---

## 🐛 Troubleshooting

### "Cannot find ConfigurationManager.psd1"

The SCCM Admin Console must be installed on the machine. The tool checks these paths:
- `E:\SCCM\AdminConsole\bin`
- `C:\Program Files (x86)\Microsoft Configuration Manager\AdminConsole\bin`
- `C:\Program Files (x86)\Microsoft Endpoint Manager\AdminConsole\bin`

If your console is installed elsewhere, build with:
```powershell
dotnet build /p:AdminConsoleBinPath="D:\YourPath\AdminConsole\bin"
```

### "Connection timed out on port 135"

Ensure the SCCM Site Server is reachable from the machine running DeployFusion on **TCP port 135 (RPC)**. This is required for the WMI/WQL connection.

### "Intune App Secret Key is missing"

The **"Use Unattended App Registration"** checkbox is enabled but no secret was saved. Go to Settings, expand the Entra fields, enter the Client Secret, and click **Save Settings**.

### "IntuneWinAppUtil exited with code 1"

Ensure the installer file path does not contain special characters. The file is copied to a temp directory first, so very long paths can also cause issues on Windows.

### Deployment Type fails to add

This uses `Add-CMScriptDeploymentType` via PowerShell, which requires the ConfigurationManager PowerShell module (part of the Admin Console). Ensure PowerShell execution policy allows script execution:
```powershell
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope LocalMachine
```

---

## 📄 License

MIT License — see [LICENSE](LICENSE) for full text.

```
Copyright (c) 2026 Mazhai Studios

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is provided
to do so, subject to the following conditions:
...
```

---

## 👤 Author

**Navaneethakrishnan Malaiyappan**  
Mazhai Studios — [https://deployfusion.in](https://deployfusion.in)  
Personal — [https://navaneethmalaiyappan.com](https://navaneethmalaiyappan.com)

---

<div align="center">

*Built with ❤️ for IT administrators who deserve better tools.*

⭐ **Star this repo** if DeployFusion saves you time!

</div>


