# WinACME Framework

WinACME Framework is a Windows-based certificate lifecycle automation solution designed to simplify the enrollment, renewal, and deployment of TLS certificates to Microsoft server applications.

The framework integrates with [**simple-acme**](https://simple-acme.com/) for ACME certificate enrollment and renewal and provides a modular deployment architecture for installing renewed certificates into supported applications.

**simple-acme is a separate project and is not developed or maintained by PKIAdvisers or Napolitano Security Consulting LLC.**

The framework uses simple-acme as its ACME client while providing additional configuration, certificate lifecycle management, diagnostics, and application-specific deployment capabilities.

## Current Release

**Version:** 1.0.0

WinACME Framework v1.0.0 is the first production release of the framework.

Production releases are distributed as digitally signed Windows installation packages through the **PKIAdvisers Product Solutions** repository.

## Documentation

The following documentation provides additional information about installing, configuring, securing, and maintaining WinACME Framework:

- [Installation Guide](./INSTALLATION.md) — System requirements, installation, initial setup, and upgrade procedures.
- [Architecture](./ARCHITECTURE.md) — Framework architecture, component relationships, certificate provisioning, deployment, automation, and security boundaries.
- [Module Reference — v1.0.0](./MODULE-REFERENCE-1.0.0.md) — Detailed reference for the PowerShell modules and functions included in WinACME Framework 1.0.0.
- [Security](./SECURITY.md) — Security model, code-signing verification, protected secrets, permissions, and operational security considerations.
- [Release Notes](./RELEASE-NOTES.md) — Release-specific changes, known limitations, fixes, and other release information.

## Key Features

WinACME Framework provides:

* ACME-based certificate enrollment and renewal using simple-acme
* Support for ACME External Account Binding (EAB)
* Interactive certificate profile configuration
* Automated renewal configuration
* Certificate deployment to supported Microsoft server applications
* Custom certificate deployment script support
* Certificate and configuration diagnostics
* Centralized configuration and logging
* Backup of framework-managed files during upgrades
* Windows Authenticode-signed production components

## Supported Integrations

### Microsoft Remote Desktop Gateway

WinACME Framework can provision certificates for Microsoft Remote Desktop Gateway and provides a custom deployment mechanism that can execute a PowerShell deployment script after certificate issuance or renewal.

Remote Desktop Gateway deployment uses the customer-provided:

```text
ImportRDSFull.ps1
```

deployment script.

`ImportRDSFull.ps1` is **not included with the WinACME Framework v1.0.0 distribution**. It is supplied separately by the customer and placed in the framework's custom deployment directory:

```text
C:\ProgramData\WinACME-Framework\Deployments\Custom
```

The framework discovers the script as a custom deployment and can invoke it as part of the certificate provisioning and renewal workflow.

Because `ImportRDSFull.ps1` is customer-provided, it remains separate from framework-owned deployment components and is not replaced by framework installation or upgrade operations.

### Microsoft SQL Server Reporting Services

WinACME Framework includes certificate deployment support for Microsoft SQL Server Reporting Services (SSRS).

The deployment process can install the renewed certificate and configure the appropriate SSRS TLS certificate bindings.

### Microsoft Exchange Hybrid

WinACME Framework includes certificate deployment support for Microsoft Exchange Hybrid environments.

The deployment process supports configuring selected Exchange services, including:

* IIS
* SMTP
* IMAP
* POP

It can also update applicable Exchange connector and Hybrid Configuration certificate references.

The initial v1.0.0 Exchange deployment implementation is intended for single-server Exchange deployments. Multi-server Exchange automation is not included in this release.

### Custom Deployments

The framework supports PowerShell-based custom certificate deployment scripts.

This allows organizations to extend certificate deployment beyond the integrations included with the framework without modifying the core application.

Customer-provided deployment scripts are maintained separately from framework-owned deployment scripts.

## Architecture

WinACME Framework separates certificate lifecycle management from application-specific certificate deployment.

At a high level:

```text
ACME Certificate Authority
          |
          v
      simple-acme
          |
          v
   WinACME Framework
          |
          +-------------------------+
          |                         |
          v                         v
Certificate Management       Deployment Scripts
                                    |
                   +----------------+----------------+
                   |                |                |
                   v                v                v
              RD Gateway           SSRS          Exchange
           Customer Script    Framework Script  Framework Script
```

This modular approach allows additional deployment integrations to be added without redesigning the certificate enrollment and renewal workflow.

## System Requirements

WinACME Framework is designed for supported Microsoft Windows Server environments.

The framework requires:

* Windows PowerShell 5.1
* Administrator privileges for installation and certificate deployment
* Network connectivity to the configured ACME service
* Access to the applicable Windows certificate store
* Appropriate administrative permissions for the target application
* [simple-acme](https://simple-acme.com/)

### simple-acme

WinACME Framework uses **simple-acme** as its ACME client for certificate enrollment and renewal.

simple-acme is a separate project and is not included as an embedded component of WinACME Framework. The framework installs and manages the ACME client separately from its own application components.

For simple-acme documentation, downloads, and additional information, visit the [official simple-acme website](https://simple-acme.com/).

Additional requirements may apply depending on the application being configured.

## Installation Location

By default, WinACME Framework installs its application and operational data beneath:

```text
C:\ProgramData\WinACME-Framework
```

The installation includes framework components and directories for configuration, logs, backups, secrets, temporary data, modules, and deployment scripts.

The ACME client is maintained separately from the framework and is not treated as an embedded framework component.

For installation details, see [INSTALLATION.md](./INSTALLATION.md).

## Configuration and Logs

Framework configuration and operational logs are maintained under:

```text
C:\ProgramData\WinACME-Framework
```

Typical directories include:

```text
C:\ProgramData\WinACME-Framework\
├── Backups\
├── Config\
├── Deployments\
│   └── Custom\
├── Logs\
├── Modules\
├── Secrets\
└── Temp\
```

This separates persistent application data from the original installation package and provides a consistent location for troubleshooting and administration.

## Custom Deployment Scripts

Organizations can extend WinACME Framework using PowerShell deployment scripts.

Custom deployment scripts are stored under:

```text
C:\ProgramData\WinACME-Framework\Deployments\Custom
```

This directory can contain both framework-distributed deployment scripts and customer-provided deployment scripts.

For example:

```text
Deployments\
└── Custom\
    ├── Install-ExchangeHybrid-Certificate.ps1
    ├── Install-SSRS-Certificate.ps1
    └── ImportRDSFull.ps1
```

In this example:

* `Install-ExchangeHybrid-Certificate.ps1` is distributed and maintained by the framework.
* `Install-SSRS-Certificate.ps1` is distributed and maintained by the framework.
* `ImportRDSFull.ps1` is customer-provided and is not included with the framework distribution.

Framework upgrades are designed to preserve customer-provided custom deployment scripts that are not part of the framework distribution.

## Security

Production WinACME Framework components are digitally signed using an Authenticode code-signing certificate issued to:

**Napolitano Security Consulting LLC**

Customers should verify the digital signature of downloaded software before installation.

For example:

```powershell
Get-AuthenticodeSignature .\WinACME-Framework-Setup.exe |
    Format-List Status, StatusMessage, SignerCertificate, TimeStamperCertificate
```

The expected signature status is:

```text
Valid
```

Release packages are also accompanied by a SHA-256 checksum that can be independently verified using:

```powershell
Get-FileHash .\WinACME-Framework-Setup.exe -Algorithm SHA256
```

For additional security and software verification information, see [SECURITY.md](./SECURITY.md).

## Support

WinACME Framework is developed and maintained as part of the PKIAdvisers certificate automation solutions portfolio.

Customers receiving the framework as part of a consulting or certificate automation engagement should use their established support channel for implementation and support requests.

## Publisher

**Napolitano Security Consulting LLC**

WinACME Framework is distributed through the **PKIAdvisers Product Solutions** repository.
