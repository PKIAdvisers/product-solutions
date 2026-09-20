# WinACME Framework Installation Guide

This document describes the installation and initial validation of WinACME Framework.

## Prerequisites

Before installing WinACME Framework, verify that the target Windows Server meets the following requirements:

* Windows PowerShell 5.1
* Local Administrator privileges
* Network connectivity to the required ACME service
* Access to the Windows Local Machine certificate store
* Administrative permissions for applications that will receive certificates
* Internet or other appropriate network access for downloading and communicating with [simple-acme](https://simple-acme.com/)

Application-specific requirements may also apply for Microsoft Remote Desktop Gateway, SQL Server Reporting Services, Microsoft Exchange, or custom deployment targets.

## Download

Download the current WinACME Framework installation package from the **Releases** section of the PKIAdvisers Product Solutions repository.

The production installation package is:

```text
WinACME-Framework-Setup.exe
```

A corresponding SHA-256 checksum file is provided with the release.

Before executing the installer, verify both the digital signature and file integrity as described below.

## Verify the Digital Signature

WinACME Framework production components are digitally signed using an Authenticode code-signing certificate issued to:

**Napolitano Security Consulting LLC**

Open Windows PowerShell and run:

```powershell
Get-AuthenticodeSignature .\WinACME-Framework-Setup.exe |
    Format-List Status, StatusMessage, SignerCertificate, TimeStamperCertificate
```

The signature status should report:

```text
Status : Valid
```

Review the `SignerCertificate` information and confirm that the software publisher is **Napolitano Security Consulting LLC**.

Do not continue with installation if the signature is invalid or the expected publisher cannot be verified.

## Verify the SHA-256 Checksum

Calculate the SHA-256 checksum of the downloaded installer:

```powershell
Get-FileHash .\WinACME-Framework-Setup.exe -Algorithm SHA256
```

Compare the resulting hash with the SHA-256 checksum published with the release.

The calculated and published values must match.

For additional software verification guidance, see [SECURITY.md](./SECURITY.md).

## Install WinACME Framework

Run:

```text
WinACME-Framework-Setup.exe
```

Administrative privileges are required.

If Windows User Account Control (UAC) requests permission to elevate the installer, review the publisher information and approve the elevation only after verifying the software.

The installer validates the framework payload before modifying the existing installation.

## Default Installation Location

WinACME Framework is installed beneath:

```text
C:\ProgramData\WinACME-Framework
```

The installed environment typically contains:

```text
C:\ProgramData\WinACME-Framework\
├── WinACME-Framework.exe
├── Backups\
├── Config\
├── Deployments\
│   └── Custom\
├── Logs\
├── Modules\
├── Secrets\
└── Temp\
```

Some directories may be created during first use rather than immediately during installation.

## Framework Components

The production distribution includes the WinACME Framework executable, supporting PowerShell modules, and framework-maintained deployment scripts.

Framework-maintained deployment scripts include:

```text
Deployments\
└── Custom\
    ├── Install-ExchangeHybrid-Certificate.ps1
    └── Install-SSRS-Certificate.ps1
```

Additional deployment scripts may be supplied separately.

For example, Remote Desktop Gateway deployments may use the customer-provided:

```text
ImportRDSFull.ps1
```

`ImportRDSFull.ps1` is not included in the WinACME Framework v1.0.0 distribution.

Customer-provided deployment scripts should be placed in:

```text
C:\ProgramData\WinACME-Framework\Deployments\Custom
```

## simple-acme

WinACME Framework uses [simple-acme](https://simple-acme.com/) as the ACME client responsible for ACME certificate enrollment and renewal.

simple-acme is a separate project and is not developed or maintained by PKIAdvisers or Napolitano Security Consulting LLC.

It is maintained separately from the WinACME Framework application components.

During framework configuration, the ACME client is installed or configured separately from the framework installation.

For simple-acme documentation and additional information, see the [official simple-acme website](https://simple-acme.com/).

## Launch WinACME Framework

After installation, launch:

```text
C:\ProgramData\WinACME-Framework\WinACME-Framework.exe
```

The framework requires administrative privileges for operations involving certificate stores, scheduled renewal tasks, application configuration, and certificate deployment.

If required, Windows will request elevation through User Account Control.

## Initial Configuration

On first use, the framework creates or initializes its operational configuration under:

```text
C:\ProgramData\WinACME-Framework
```

Configuration data is maintained separately from the executable and framework modules.

Depending on the intended certificate workflow, configuration may include:

* ACME service configuration
* External Account Binding (EAB) information
* Certificate subject information
* Subject Alternative Names (SANs)
* Certificate storage configuration
* Renewal configuration
* Deployment script selection
* Application-specific deployment options

Sensitive information used by the framework is maintained separately from the general configuration wherever supported.

## ACME Service Configuration

The framework supports ACME services that require External Account Binding.

Customers should obtain the required ACME account information from their Certificate Authority or ACME service provider.

Depending on the service, this may include:

* ACME directory URL
* EAB Key Identifier
* EAB HMAC key

Treat ACME credentials and EAB secrets as sensitive information.

Do not place credentials or secret values in public repositories, documentation, tickets, or other unsecured locations.

## Custom Deployment Scripts

Customer-provided deployment scripts should be copied to:

```text
C:\ProgramData\WinACME-Framework\Deployments\Custom
```

The framework discovers supported PowerShell scripts from its deployment directories and makes them available to the applicable certificate deployment workflow.

Customer-provided scripts are separate from framework-maintained components.

Framework installation and upgrade operations are designed to preserve customer-provided deployment scripts that are not included in the framework distribution.

## Validate the Installation

After installation, confirm that the primary executable exists:

```powershell
Test-Path 'C:\ProgramData\WinACME-Framework\WinACME-Framework.exe'
```

Expected result:

```text
True
```

Verify its Authenticode signature:

```powershell
Get-AuthenticodeSignature 'C:\ProgramData\WinACME-Framework\WinACME-Framework.exe' |
    Format-List Status, StatusMessage, SignerCertificate
```

The expected status is:

```text
Valid
```

You can also verify the signatures of the installed PowerShell modules:

```powershell
Get-ChildItem 'C:\ProgramData\WinACME-Framework\Modules\*.psm1' |
    Get-AuthenticodeSignature |
    Select-Object Path, Status
```

Production framework modules should report:

```text
Valid
```

Framework-maintained deployment scripts can be checked with:

```powershell
Get-ChildItem 'C:\ProgramData\WinACME-Framework\Deployments\Custom\*.ps1' |
    Get-AuthenticodeSignature |
    Select-Object Path, Status
```

Customer-provided scripts may have a different signing status depending on how they are distributed and maintained by the customer.

## Logs

Framework logs are stored under:

```text
C:\ProgramData\WinACME-Framework\Logs
```

Review these logs when troubleshooting certificate provisioning, renewal, configuration, or deployment operations.

## Upgrades

A newer WinACME Framework installation package can be used to update an existing installation.

Before replacing framework-managed files, the installer creates a backup beneath:

```text
C:\ProgramData\WinACME-Framework\Backups
```

Framework-owned components may be replaced during an upgrade.

Customer-provided deployment scripts that are not included in the framework distribution are preserved.

Customers should still maintain appropriate system backups and change-control procedures before performing production upgrades.

## Uninstallation

WinACME Framework v1.0.0 does not rely on a traditional Windows application installation model that automatically removes all operational data.

Before removing the framework, review:

* Active certificate renewal configuration
* Scheduled renewal tasks
* ACME client configuration
* Certificate deployment scripts
* Framework configuration
* Secrets
* Logs and backups

Do not remove the framework from a production server until any certificate renewal processes that depend on it have been identified and appropriately migrated or disabled.

## Next Steps

After installation:

1. Launch WinACME Framework as Administrator.
2. Install or verify the simple-acme ACME client.
3. Configure the required ACME service.
4. Configure the certificate profile.
5. Select the appropriate certificate deployment method.
6. Provision and deploy the certificate.
7. Verify the target application is presenting or using the expected certificate.
8. Verify automated renewal configuration.

Certificate deployment should be tested in a non-production environment before implementing automated renewal and deployment on production systems.

## Additional Documentation

* [WinACME Framework Overview](./README.md)
* [Release Notes](./RELEASE-NOTES.md)
* [Security and Software Verification](./SECURITY.md)

For simple-acme documentation, visit the [official simple-acme website](https://simple-acme.com/).
