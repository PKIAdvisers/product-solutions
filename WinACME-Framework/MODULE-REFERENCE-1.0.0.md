# WinACME-Framework 1.0.0 Module Reference

> Source baseline: Release 1.0.0 PowerShell source supplied for documentation. This document describes the code as implemented in that baseline.

## 1. Architecture summary

The framework is a Windows PowerShell 5.1 administrative application. `Start-WinACME-Framework.ps1` enforces elevation, imports the runtime modules, validates required commands, initializes `C:\ProgramData\WinACME-Framework`, initializes the protected secrets store, and enters a 12-option interactive menu.

### Runtime module load order

```text
Logging → Configuration → Secrets → Menu → Installer → Wizard → Deployment
        → ExchangeHybrid → Provisioning → Diagnostics → Automation
```

SSRS is a deployment-support module in the supplied 1.0.0 source set; it is not imported directly by the main entry-point script.

### Main menu dispatch

| Menu | Operation | Entry function |
|---:|---|---|
| 1 | Install / upgrade simple-acme | `Install-WinAcme` |
| 2 | Configure certificate profile | `Invoke-CertificateProfileWizard` |
| 3 | Configure certificate storage | `Invoke-CertificateStorageWizard` |
| 4 | Configure renewal policy | `Invoke-RenewalPolicyWizard` |
| 5 | Select deployment script | `Select-DeploymentScript` |
| 6 | Configure Exchange Hybrid | `Invoke-ExchangeHybridConfigurationWizard` |
| 7 | Show configuration | `Show-Configuration` |
| 8 | Preview simple-acme command | `Show-WinAcmeCommandPreview` |
| 9 | Provision certificate | `Invoke-WinAcmeProvisioning` |
| 10 | Diagnostics | `Show-FrameworkDiagnostics` |
| 11 | Remove automation | `Invoke-RemoveAutomation` |
| 12 | Exit | Main loop |

## 2. Module inventory

| Module | Functions | Exported | Primary responsibility |
|---|---:|---:|---|
| `Logging.psm1` | 2 | 2 | Framework logging. |
| `Configuration.psm1` | 13 | 6 | Runtime directories, configuration schema/defaults, persistence, and display. |
| `Secrets.psm1` | 9 | 6 | DPAPI-protected secret persistence and NTFS ACL hardening. |
| `Menu.psm1` | 2 | 2 | Interactive console menu. |
| `Installer.psm1` | 6 | 6 | simple-acme package selection, validation, backup, and installation/upgrade. |
| `Wizard.psm1` | 10 | 5 | Interactive certificate, storage, renewal, and Exchange Hybrid configuration. |
| `Deployment.psm1` | 7 | 7 | Built-in/custom deployment-script discovery and selection. |
| `ExchangeHybrid.psm1` | 10 | 10 | Exchange Hybrid readiness, baseline capture, and post-deployment validation. |
| `Provisioning.psm1` | 17 | 14 | simple-acme settings, argument construction, validation, command preview, and issuance orchestration. |
| `Diagnostics.psm1` | 9 | 9 | Structured health/readiness checks across framework subsystems. |
| `Automation.psm1` | 12 | 12 | simple-acme renewal and scheduled-task discovery/removal. |
| `SSRS.psm1` | 13 | 12 | SSRS WMI discovery, certificate validation, private-key ACLs, and SSL bindings. |

**Total:** 110 functions. Functions not exported are marked **Internal** below.

## 3.1 `Logging.psm1`

Framework logging.

| Function | Scope | Parameters | Purpose | Framework dependencies |
|---|---|---|---|---|
| `Initialize-Logging` | Exported | `-Path` | Initializes the framework log directory and current daily log file. | — |
| `Write-Log` | Exported | `-Message`, `-Level` | Writes a timestamped framework log entry at the requested severity level. | — |

## 3.2 `Configuration.psm1`

Runtime directories, configuration schema/defaults, persistence, and display.

| Function | Scope | Parameters | Purpose | Framework dependencies |
|---|---|---|---|---|
| `Get-ApplicationRoot` | Exported | — | Returns the initialized persistent framework root path. | — |
| `Get-ApplicationPaths` | Exported | — | Returns the framework runtime directory map beneath ApplicationRoot. | — |
| `Test-ObjectProperty` | **Internal** | `-InputObject`, `-PropertyName` | Tests whether a PowerShell object exposes a named property. | — |
| `Add-ConfigurationProperty` | **Internal** | `-InputObject`, `-PropertyName`, `-Value` | Adds a missing configuration property without overwriting an existing value. | `Test-ObjectProperty` |
| `New-DefaultCertificateProfile` | **Internal** | — | Creates the default certificate-profile configuration object. | — |
| `New-DefaultStorageConfiguration` | **Internal** | — | Creates the default certificate-storage configuration object. | — |
| `New-DefaultDeploymentConfiguration` | **Internal** | — | Creates the default deployment configuration, including Exchange Hybrid defaults. | — |
| `New-DefaultRenewalConfiguration` | **Internal** | — | Creates the default certificate-validity and renewal-policy configuration. | — |
| `Initialize-Application` | Exported | `-RootPath` | Creates runtime directories, initializes logging, creates/loads config.json, and migrates missing configuration properties to the current schema. | `Initialize-Logging`, `Write-Log`, `Test-ObjectProperty`, `Add-ConfigurationProperty`, `New-DefaultCertificateProfile`, `New-DefaultStorageConfiguration`, `New-DefaultDeploymentConfiguration`, `New-DefaultRenewalConfiguration`, `Save-ApplicationConfiguration` |
| `Get-ApplicationConfiguration` | Exported | — | Returns the in-memory application configuration. | — |
| `Get-DisplayValue` | **Internal** | `-Value`, `-DefaultValue` | Normalizes values for safe human-readable configuration display. | — |
| `Show-Configuration` | Exported | — | Displays the current framework configuration while reporting secret presence without exposing secret values. | `Get-ApplicationConfiguration`, `Get-DisplayValue`, `Test-Secret` |
| `Save-ApplicationConfiguration` | Exported | `-Config` | Persists the supplied configuration to config.json and refreshes in-memory state. | — |

## 3.3 `Secrets.psm1`

DPAPI-protected secret persistence and NTFS ACL hardening.

| Function | Scope | Parameters | Purpose | Framework dependencies |
|---|---|---|---|---|
| `Get-SecretsRoot` | **Internal** | — | Returns the framework Secrets directory. | `Get-ApplicationPaths` |
| `Get-SecretPath` | Exported | `-Name` | Resolves the protected file path for a named framework secret. | `Get-SecretsRoot` |
| `Set-SecretDirectoryAcl` | **Internal** | `-Path` | Applies restrictive NTFS permissions to a secret directory. | — |
| `Set-SecretFileAcl` | **Internal** | `-Path` | Applies restrictive NTFS permissions to a secret file. | — |
| `Initialize-Secrets` | Exported | — | Creates/secures the secrets store and records initialization. | `Write-Log`, `Get-SecretsRoot`, `Set-SecretDirectoryAcl` |
| `Save-Secret` | Exported | `-Name`, `-SecureString` | DPAPI-protects and saves a SecureString secret, then restricts its file ACL. | `Write-Log`, `Get-SecretPath`, `Set-SecretFileAcl` |
| `Get-Secret` | Exported | `-Name` | Loads and decrypts a named DPAPI-protected secret as a SecureString. | `Get-SecretPath` |
| `Test-Secret` | Exported | `-Name` | Tests whether a named protected secret exists. | `Get-SecretPath` |
| `Remove-Secret` | Exported | `-Name` | Deletes a named protected secret and logs the operation. | `Write-Log`, `Get-SecretPath` |

## 3.4 `Menu.psm1`

Interactive console menu.

| Function | Scope | Parameters | Purpose | Framework dependencies |
|---|---|---|---|---|
| `Show-MainMenu` | Exported | — | Renders the interactive framework main menu and current version/context. | `Get-ApplicationConfiguration` |
| `Wait-ForUser` | Exported | — | Pauses menu execution until the operator acknowledges the prompt. | — |

## 3.5 `Installer.psm1`

simple-acme package selection, validation, backup, and installation/upgrade.

| Function | Scope | Parameters | Purpose | Framework dependencies |
|---|---|---|---|---|
| `Select-WinAcmeZip` | Exported | — | Prompts the operator to select a simple-acme ZIP package. | — |
| `Test-WinAcmeZip` | Exported | `-ZipFile` | Validates that the selected ZIP is suitable for framework installation. | `Write-Log` |
| `Get-WinAcmeVersionFromPath` | Exported | `-Path` | Determines the simple-acme version from an extracted installation path. | — |
| `Backup-WinAcme` | Exported | `-Path` | Creates a timestamped backup of an existing simple-acme installation. | `Write-Log`, `Get-ApplicationPaths` |
| `Get-WinAcmeVersionFromZip` | Exported | `-ZipFile` | Extracts/inspects a ZIP package to determine its simple-acme version. | `Get-WinAcmeVersionFromPath` |
| `Install-WinAcme` | Exported | — | Coordinates selection, validation, backup, extraction, version detection, and configuration update for simple-acme installation/upgrade. | `Write-Log`, `Get-ApplicationConfiguration`, `Save-ApplicationConfiguration`, `Select-WinAcmeZip`, `Test-WinAcmeZip`, `Backup-WinAcme`, `Get-WinAcmeVersionFromZip` |

## 3.6 `Wizard.psm1`

Interactive certificate, storage, renewal, and Exchange Hybrid configuration.

| Function | Scope | Parameters | Purpose | Framework dependencies |
|---|---|---|---|---|
| `Read-ProfileString` | **Internal** | `-Prompt`, `-CurrentValue`, `-Required` | Prompts for a profile string while supporting current/default values and required input. | — |
| `Read-YesNo` | **Internal** | `-Prompt`, `-CurrentValue` | Prompts for and validates a yes/no configuration choice. | — |
| `Test-AcmeDirectoryUrl` | **Internal** | `-Url` | Validates the configured ACME directory URL. | — |
| `Test-EmailAddress` | **Internal** | `-EmailAddress` | Performs format validation on the configured account email address. | — |
| `Test-CertificateSubject` | **Internal** | `-Subject` | Validates the configured certificate subject. | — |
| `Invoke-CertificateProfileWizard` | Exported | — | Interactively configures ACME account/EAB and certificate profile settings, storing the HMAC secret separately. | `Write-Log`, `Get-ApplicationConfiguration`, `Save-ApplicationConfiguration`, `Save-Secret`, `Test-Secret`, `Read-ProfileString`, `Read-YesNo`, `Test-AcmeDirectoryUrl`, `Test-EmailAddress`, `Test-CertificateSubject` |
| `Invoke-CertificateStorageWizard` | Exported | — | Interactively configures certificate storage options and paths. | `Get-ApplicationPaths`, `Get-ApplicationConfiguration`, `Save-ApplicationConfiguration`, `Read-ProfileString` |
| `Get-CalculatedRenewalDays` | Exported | `-CertificateValidityDays`, `-RenewBeforeExpirationDays` | Calculates the simple-acme renewal interval from certificate validity and renew-before-expiration values. | — |
| `Invoke-RenewalPolicyWizard` | Exported | — | Interactively configures renewal policy and synchronizes the calculated renewal interval to simple-acme settings. | `Write-Log`, `Get-ApplicationConfiguration`, `Save-ApplicationConfiguration`, `Get-CalculatedRenewalDays`, `Set-WinAcmeRenewalDays` |
| `Invoke-ExchangeHybridConfigurationWizard` | Exported | — | Interactively configures Exchange Hybrid deployment options. | `Write-Log`, `Get-ApplicationConfiguration`, `Save-ApplicationConfiguration`, `Read-YesNo` |

## 3.7 `Deployment.psm1`

Built-in/custom deployment-script discovery and selection.

| Function | Scope | Parameters | Purpose | Framework dependencies |
|---|---|---|---|---|
| `Get-DeploymentRoot` | Exported | — | Returns the framework Deployments directory. | `Get-ApplicationRoot` |
| `Get-BuiltInDeploymentScripts` | Exported | — | Enumerates deployment scripts shipped with the framework. | `Get-DeploymentRoot` |
| `Get-CustomDeploymentScripts` | Exported | — | Enumerates customer-provided deployment scripts. | `Get-DeploymentRoot` |
| `Get-DeploymentScripts` | Exported | — | Returns the combined built-in and custom deployment-script inventory. | `Get-BuiltInDeploymentScripts`, `Get-CustomDeploymentScripts` |
| `Disable-DeploymentSelection` | Exported | — | Disables deployment in configuration and clears the selected deployment metadata. | `Get-ApplicationConfiguration`, `Save-ApplicationConfiguration` |
| `Select-DeploymentScript` | Exported | — | Presents available deployment scripts and persists the operator selection. | `Get-ApplicationConfiguration`, `Save-ApplicationConfiguration`, `Get-DeploymentScripts`, `Disable-DeploymentSelection` |
| `Get-SelectedDeployment` | Exported | — | Returns the currently configured deployment selection. | `Get-ApplicationConfiguration` |

## 3.8 `ExchangeHybrid.psm1`

Exchange Hybrid readiness, baseline capture, and post-deployment validation.

| Function | Scope | Parameters | Purpose | Framework dependencies |
|---|---|---|---|---|
| `Get-ExchangeHybridServerFqdn` | Exported | — | Determines the local Exchange server FQDN used for remote Exchange management. | — |
| `New-ExchangeHybridSession` | Exported | — | Creates a PowerShell remoting session to the Exchange management endpoint. | `Write-Log`, `Get-ExchangeHybridServerFqdn` |
| `Get-ExchangeHybridRemoteState` | Exported | `-Session` | Queries relevant Exchange certificate/service state through an existing session. | — |
| `Test-ExchangeHybridPrerequisites` | Exported | — | Runs Exchange Hybrid pre-deployment checks and returns structured readiness results. | `Write-Log`, `Get-ExchangeHybridServerFqdn`, `New-ExchangeHybridSession`, `Get-ExchangeHybridRemoteState` |
| `Write-ExchangeHybridBaselineLog` | Exported | `-PreflightResult` | Logs the pre-deployment Exchange Hybrid baseline and target certificate names. | `Write-Log`, `Get-ApplicationConfiguration`, `Get-CertificateHostNames` |
| `Show-ExchangeHybridPreflight` | Exported | — | Runs and displays the Exchange Hybrid preflight assessment. | `Get-ApplicationConfiguration`, `Test-ExchangeHybridPrerequisites`, `Write-ExchangeHybridBaselineLog` |
| `Get-ExchangeHybridCertificateSnapshot` | Exported | `-StoreName` | Captures certificate thumbprints before provisioning so the newly issued certificate can be identified. | — |
| `Get-NewExchangeHybridCertificate` | Exported | `-PreviousThumbprints`, `-StoreName` | Finds the certificate added after provisioning by comparing current state with the prior snapshot. | — |
| `Test-ExchangeHybridPostDeployment` | Exported | `-Thumbprint` | Validates the newly deployed certificate and configured Exchange services after provisioning. | `Write-Log`, `Get-ApplicationConfiguration`, `New-ExchangeHybridSession` |
| `Show-ExchangeHybridPostDeployment` | Exported | `-ValidationResult`, `-Thumbprint` | Displays structured Exchange Hybrid post-deployment validation results. | — |

## 3.9 `Provisioning.psm1`

simple-acme settings, argument construction, validation, command preview, and issuance orchestration.

| Function | Scope | Parameters | Purpose | Framework dependencies |
|---|---|---|---|---|
| `Get-WinAcmeExecutable` | Exported | — | Resolves and validates the configured wacs.exe path. | `Get-ApplicationConfiguration` |
| `Get-WinAcmeSettingsPath` | Exported | — | Resolves the simple-acme settings.json path used by the framework. | `Get-ApplicationConfiguration` |
| `Save-WinAcmeSettings` | **Internal** | `-Settings`, `-SettingsPath` | Writes a modified simple-acme settings object back to settings.json. | — |
| `Set-WinAcmePrivateKeyExportableSetting` | Exported | `-Enabled` | Updates simple-acme settings to control private-key exportability. | `Write-Log`, `Get-WinAcmeSettingsPath`, `Save-WinAcmeSettings` |
| `Set-WinAcmeRenewalDays` | Exported | `-RenewalDays` | Updates the simple-acme renewal-days setting to match framework policy. | `Write-Log`, `Get-WinAcmeSettingsPath`, `Save-WinAcmeSettings` |
| `Get-SelectedDeploymentScriptPath` | Exported | — | Resolves and validates the configured deployment script path. | `Get-ApplicationRoot`, `Get-ApplicationConfiguration` |
| `Get-WinAcmeStorageArguments` | Exported | — | Builds simple-acme command-line arguments for the selected certificate storage mode. | `Get-ApplicationConfiguration` |
| `Get-WinAcmeDeploymentArguments` | Exported | — | Builds simple-acme installation/deployment arguments for the selected deployment script. | `Get-ApplicationConfiguration`, `Get-SelectedDeploymentScriptPath` |
| `Test-CertificateDnsName` | Exported | `-DnsName` | Validates a configured certificate DNS name. | — |
| `Test-ProvisioningConfiguration` | Exported | `-IncludeEnvironmentPreflight` | Performs configuration and optional environmental preflight validation before certificate issuance. | `Get-ApplicationConfiguration`, `Test-Secret`, `Test-ExchangeHybridPrerequisites`, `Get-WinAcmeExecutable`, `Get-SelectedDeploymentScriptPath`, `Get-WinAcmeStorageArguments`, `Test-CertificateDnsName`, `Get-CertificateHostNames` |
| `Show-DomainAuthorizationNotice` | Exported | — | Displays the framework notice concerning pre-authorized/pre-validated domains. | — |
| `Get-WinAcmeProvisioningArguments` | **Internal** | `-ClearEabHmac` | Builds the complete simple-acme provisioning argument list, including ACME/EAB, identifiers, storage, renewal, and deployment options. | `Get-ApplicationConfiguration`, `Get-WinAcmeStorageArguments`, `Get-WinAcmeDeploymentArguments`, `Get-CertificateHostNames` |
| `Get-SanitizedWinAcmeCommand` | Exported | — | Builds a display-safe simple-acme command preview with sensitive EAB material omitted/redacted. | `Get-ApplicationConfiguration`, `Get-WinAcmeExecutable`, `Get-SelectedDeploymentScriptPath`, `Get-CertificateHostNames` |
| `Show-ProvisioningValidationErrors` | **Internal** | `-ValidationResult` | Displays structured provisioning validation failures to the operator. | — |
| `Show-WinAcmeCommandPreview` | Exported | — | Validates configuration and displays the sanitized command that provisioning would execute. | `Get-ApplicationConfiguration`, `Test-ProvisioningConfiguration`, `Show-DomainAuthorizationNotice`, `Get-SanitizedWinAcmeCommand`, `Show-ProvisioningValidationErrors` |
| `Invoke-WinAcmeProvisioning` | Exported | — | Coordinates validation, EAB secret retrieval, simple-acme settings, certificate issuance, deployment, logging, and Exchange Hybrid pre/post checks when selected. | `Write-Log`, `Get-ApplicationConfiguration`, `Get-Secret`, `Show-ExchangeHybridPreflight`, `Get-ExchangeHybridCertificateSnapshot`, `Get-NewExchangeHybridCertificate`, `Test-ExchangeHybridPostDeployment`, `Show-ExchangeHybridPostDeployment`, `Get-WinAcmeExecutable`, `Set-WinAcmePrivateKeyExportableSetting`, `Set-WinAcmeRenewalDays`, `Test-ProvisioningConfiguration`, `Show-DomainAuthorizationNotice`, `Get-WinAcmeProvisioningArguments`, `Get-SanitizedWinAcmeCommand`, `Show-ProvisioningValidationErrors`, `Get-CertificateHostNames` |
| `Get-CertificateHostNames` | Exported | — | Returns the configured certificate host names derived from Subject and SAN configuration. | `Get-ApplicationConfiguration` |

## 3.10 `Diagnostics.psm1`

Structured health/readiness checks across framework subsystems.

| Function | Scope | Parameters | Purpose | Framework dependencies |
|---|---|---|---|---|
| `New-DiagnosticResult` | Exported | `-Category`, `-Check`, `-Status`, `-Message` | Creates a standardized diagnostic result object with category, check, status, and message. | — |
| `Test-FrameworkDiagnostics` | Exported | — | Checks framework directories, configuration, secrets, and core runtime prerequisites. | `Get-ApplicationRoot`, `Test-Secret`, `New-DiagnosticResult` |
| `Test-WinAcmeDiagnostics` | Exported | — | Checks simple-acme installation/configuration health and related framework settings. | `Write-Log`, `Get-ApplicationRoot`, `Get-ApplicationConfiguration`, `New-DiagnosticResult` |
| `Test-CertificateProfileDiagnostics` | Exported | — | Checks certificate profile, EAB secret availability, identifiers, and storage-related configuration. | `Get-ApplicationConfiguration`, `Test-Secret`, `Get-WinAcmeStorageArguments`, `Get-CertificateHostNames`, `New-DiagnosticResult` |
| `Test-AutomationDiagnostics` | Exported | — | Checks the Windows scheduled-task/automation state used by simple-acme. | `New-DiagnosticResult` |
| `Test-DeploymentDiagnostics` | Exported | — | Checks deployment selection, script availability, and deployment-related provisioning readiness. | `Get-ApplicationConfiguration`, `Get-SelectedDeploymentScriptPath`, `Test-ProvisioningConfiguration`, `New-DiagnosticResult` |
| `Get-FrameworkDiagnostics` | Exported | — | Runs all diagnostic categories and returns the combined result set. | `Test-FrameworkDiagnostics`, `Test-WinAcmeDiagnostics`, `Test-CertificateProfileDiagnostics`, `Test-AutomationDiagnostics`, `Test-DeploymentDiagnostics` |
| `Write-DiagnosticsLog` | Exported | `-Results` | Writes diagnostic results to the framework log. | `Write-Log` |
| `Show-FrameworkDiagnostics` | Exported | — | Runs, displays, and logs the complete framework diagnostic report. | `Get-FrameworkDiagnostics`, `Write-DiagnosticsLog` |

## 3.11 `Automation.psm1`

simple-acme renewal and scheduled-task discovery/removal.

| Function | Scope | Parameters | Purpose | Framework dependencies |
|---|---|---|---|---|
| `Get-WinAcmeAutomationExecutable` | Exported | — | Resolves and validates the simple-acme executable for automation-management operations. | `Get-ApplicationConfiguration` |
| `Get-WinAcmeRenewalFiles` | Exported | — | Discovers simple-acme *.renewal.json files in standard and configured locations. | `Write-Log`, `Get-ApplicationConfiguration` |
| `Get-WinAcmeRenewals` | Exported | — | Parses discovered renewal files into a normalized renewal inventory. | `Write-Log`, `Get-WinAcmeRenewalFiles` |
| `Get-WinAcmeScheduledTasks` | Exported | — | Discovers simple-acme-related Windows scheduled tasks. | — |
| `Show-WinAcmeRenewals` | Exported | `-Renewals` | Displays a supplied simple-acme renewal inventory. | — |
| `Show-WinAcmeScheduledTasks` | Exported | `-Tasks` | Displays a supplied simple-acme scheduled-task inventory. | — |
| `Invoke-WinAcmeRenewalCancellation` | Exported | `-RenewalId`, `-FriendlyName` | Invokes simple-acme cancellation for a selected renewal identifier. | `Write-Log`, `Get-ApplicationConfiguration`, `Get-WinAcmeAutomationExecutable` |
| `Remove-WinAcmeSelectedRenewal` | Exported | — | Interactively selects and cancels one simple-acme renewal. | `Get-WinAcmeRenewals`, `Show-WinAcmeRenewals`, `Invoke-WinAcmeRenewalCancellation` |
| `Remove-AllWinAcmeRenewals` | Exported | — | Cancels all discovered simple-acme renewals after operator confirmation. | `Get-WinAcmeRenewals`, `Show-WinAcmeRenewals`, `Invoke-WinAcmeRenewalCancellation` |
| `Remove-WinAcmeScheduledTasks` | Exported | — | Removes discovered simple-acme scheduled tasks after operator confirmation. | `Write-Log`, `Get-WinAcmeScheduledTasks`, `Show-WinAcmeScheduledTasks` |
| `Show-RemoveAutomationSafetyNotice` | Exported | — | Displays safety guidance before destructive automation removal. | — |
| `Invoke-RemoveAutomation` | Exported | — | Coordinates discovery, display, selective/all renewal cancellation, scheduled-task removal, and post-removal diagnostics. | `Show-FrameworkDiagnostics`, `Get-WinAcmeRenewals`, `Get-WinAcmeScheduledTasks`, `Show-WinAcmeRenewals`, `Show-WinAcmeScheduledTasks`, `Remove-WinAcmeSelectedRenewal`, `Remove-AllWinAcmeRenewals`, `Remove-WinAcmeScheduledTasks`, `Show-RemoveAutomationSafetyNotice` |

## 3.12 `SSRS.psm1`

SSRS WMI discovery, certificate validation, private-key ACLs, and SSL bindings.

| Function | Scope | Parameters | Purpose | Framework dependencies |
|---|---|---|---|---|
| `Get-SSRSAdminNamespace` | Exported | — | Discovers the installed SSRS WMI Admin namespace. | — |
| `Get-SSRSConfigurationObject` | Exported | — | Returns the SSRS WMI configuration object used for binding operations. | `Get-SSRSAdminNamespace` |
| `Get-SSRSConfiguration` | Exported | — | Returns normalized SSRS instance/service configuration information. | `Get-SSRSAdminNamespace`, `Get-SSRSConfigurationObject` |
| `Get-SSRSCertificateBindings` | Exported | `-LCID` | Enumerates current SSRS SSL certificate bindings. | `Get-SSRSConfigurationObject` |
| `Get-SSRSAvailableCertificates` | Exported | — | Enumerates certificates available to SSRS for TLS binding. | `Get-SSRSConfigurationObject` |
| `Test-SSRSCertificate` | Exported | `-Thumbprint` | Validates that a specified certificate is suitable for SSRS deployment. | — |
| `Resolve-CNGPrivateKeyPath` | Exported | `-UniqueName` | Resolves the filesystem path of a CNG private key from its unique name. | — |
| `Get-CertificatePrivateKeyPath` | Exported | `-Certificate` | Determines the backing private-key file for a certificate. | `Resolve-CNGPrivateKeyPath` |
| `Grant-CertificatePrivateKeyReadAccess` | Exported | `-Certificate`, `-Identity` | Grants the SSRS service identity read access to the certificate private key. | `Get-CertificatePrivateKeyPath` |
| `Set-SSRSCertificateBindings` | Exported | `-Thumbprint`, `-IPAddress`, `-Port`, `-LCID` | Creates SSRS SSL certificate bindings for the requested certificate/IP/port. | `Get-SSRSConfigurationObject`, `Test-SSRSCertificate` |
| `Remove-SSRSCertificateBindings` | Exported | `-LCID` | Removes existing SSRS SSL certificate bindings. | `Get-SSRSConfigurationObject`, `Get-SSRSCertificateBindings` |
| `Test-SSRSCertificateBinding` | Exported | `-Thumbprint`, `-LCID` | Verifies that SSRS is bound to the expected certificate. | `Get-SSRSCertificateBindings` |
| `Install-SSRSCertificate` | **Internal** | `-Thumbprint`, `-LCID` | Coordinates SSRS certificate validation, service-account private-key access, and SSL binding deployment. | `Get-SSRSConfigurationObject`, `Get-SSRSConfiguration`, `Get-SSRSCertificateBindings`, `Test-SSRSCertificate`, `Grant-CertificatePrivateKeyReadAccess`, `Set-SSRSCertificateBindings` |

## 4. High-level dependency map

```text
Start-WinACME-Framework.ps1
│
├─ Configuration ──► Logging
│       ├────────────► Secrets
│       └────────────► shared configuration consumers
│
├─ Wizard ──────────► Configuration / Secrets / Provisioning settings
├─ Deployment ──────► Configuration
├─ ExchangeHybrid ──► Configuration / Provisioning helpers / Logging
├─ Provisioning ────► Configuration / Secrets / Deployment / ExchangeHybrid / Logging
├─ Diagnostics ─────► Configuration / Secrets / Provisioning / Deployment / Logging
└─ Automation ──────► Configuration / Diagnostics / Logging

SSRS.psm1 ─────────► SSRS WMI + Windows certificate/private-key facilities
```

## 5. Key orchestration functions

| Function | Why it matters |
|---|---|
| `Initialize-Application` | Establishes the persistent runtime environment and upgrades the configuration object to the current schema. |
| `Install-WinAcme` | Owns the simple-acme installation/upgrade lifecycle. |
| `Invoke-CertificateProfileWizard` | Captures certificate/ACME/EAB configuration while separating the HMAC secret from config.json. |
| `Select-DeploymentScript` | Connects certificate issuance to the selected built-in or custom deployment behavior. |
| `Test-ProvisioningConfiguration` | Acts as the principal readiness gate before issuance. |
| `Invoke-WinAcmeProvisioning` | Central issuance orchestrator; combines validation, secrets, settings, simple-acme execution, deployment, and optional Exchange Hybrid validation. |
| `Show-FrameworkDiagnostics` | Aggregates and presents framework health checks. |
| `Invoke-RemoveAutomation` | Coordinates destructive removal of renewals and scheduled tasks with safety prompts. |
| `Install-SSRSCertificate` | SSRS-specific deployment orchestrator for validation, private-key access, and bindings. |

## 6. Documentation notes for 2.0.0 planning

This 1.0.0 baseline exposes several architectural boundaries that should be preserved explicitly in later migration documentation: configuration/schema management, secret storage, simple-acme installation, certificate-profile configuration, provisioning/issuance, deployment selection, product-specific deployment logic, diagnostics, and automation cleanup. Release 2.0.0 changes can be mapped against these boundaries to distinguish compatible behavior from refactored or replaced behavior.
