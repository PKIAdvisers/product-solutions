# WinACME Framework Release Notes

This document contains release information for production versions of WinACME Framework distributed through the PKIAdvisers Product Solutions repository.

---

# Version 1.0.0

**Release:** Initial Production Release
**Status:** Production

WinACME Framework v1.0.0 is the first production release of the framework.

This release provides a modular Windows-based platform for ACME certificate enrollment, renewal, certificate lifecycle management, and application-specific certificate deployment using [simple-acme](https://simple-acme.com/) as the underlying ACME client.

## Highlights

Version 1.0.0 introduces:

* ACME certificate enrollment and renewal using simple-acme
* External Account Binding (EAB) support
* Interactive certificate profile configuration
* Certificate renewal configuration
* Windows certificate store integration
* Automated certificate deployment
* Microsoft SQL Server Reporting Services deployment support
* Microsoft Exchange Hybrid deployment support
* Custom PowerShell deployment script support
* Microsoft Remote Desktop Gateway deployment through a customer-provided deployment script
* Certificate and configuration diagnostics
* Renewal automation management
* Centralized configuration and logging
* Framework file backup during upgrades
* Authenticode-signed production components

## ACME Client

WinACME Framework v1.0.0 uses [simple-acme](https://simple-acme.com/) for ACME certificate enrollment and renewal.

simple-acme is a separate project and is not developed or maintained by PKIAdvisers or Napolitano Security Consulting LLC.

The ACME client is maintained separately from the WinACME Framework application components.

For simple-acme documentation and project information, visit the [official simple-acme website](https://simple-acme.com/).

## Microsoft SQL Server Reporting Services

Version 1.0.0 includes framework-maintained certificate deployment support for Microsoft SQL Server Reporting Services (SSRS).

The framework distribution includes:

```text
Install-SSRS-Certificate.ps1
```

The deployment integration supports installing a renewed certificate and configuring the applicable SSRS TLS certificate bindings.

Application-specific permissions and configuration requirements still apply.

## Microsoft Exchange Hybrid

Version 1.0.0 includes framework-maintained certificate deployment support for Microsoft Exchange Hybrid environments.

The framework distribution includes:

```text
Install-ExchangeHybrid-Certificate.ps1
```

The deployment integration supports selected Exchange services including:

* IIS
* SMTP
* IMAP
* POP

The deployment workflow can also update applicable Exchange Send Connector, Receive Connector, and Hybrid Configuration certificate references.

### Exchange v1.0.0 Scope

The Exchange Hybrid implementation included with v1.0.0 is intended for a **single on-premises Exchange server** deployment.

Multi-server Exchange certificate deployment and Exchange relay-server automation are not included in v1.0.0.

These environments require additional deployment planning and should not be assumed to be supported by the initial Exchange integration.

## Microsoft Remote Desktop Gateway

WinACME Framework supports Remote Desktop Gateway certificate deployment through its custom deployment-script architecture.

Remote Desktop Gateway deployments may use:

```text
ImportRDSFull.ps1
```

`ImportRDSFull.ps1` is **customer-provided and is not included with the WinACME Framework v1.0.0 distribution**.

The script is placed in:

```text
C:\ProgramData\WinACME-Framework\Deployments\Custom
```

and can then be selected as a custom deployment within the framework.

Because the script is customer-provided, it remains outside the framework-maintained software distribution and signing boundary.

## Custom Deployment Scripts

Version 1.0.0 supports PowerShell-based custom certificate deployment scripts.

Custom deployment scripts are located under:

```text
C:\ProgramData\WinACME-Framework\Deployments\Custom
```

This directory can contain both framework-maintained and customer-provided deployment scripts.

The v1.0.0 framework distribution provides:

```text
Install-ExchangeHybrid-Certificate.ps1
Install-SSRS-Certificate.ps1
```

Customer-provided scripts that are not part of the framework distribution are designed to be preserved during framework installation and upgrade operations.

## Installation

The production installer is distributed as:

```text
WinACME-Framework-Setup.exe
```

The default framework installation location is:

```text
C:\ProgramData\WinACME-Framework
```

The installed environment includes the framework executable, PowerShell modules, deployment scripts, and operational directories for configuration, logging, backups, secrets, and temporary data.

See [INSTALLATION.md](./INSTALLATION.md) for installation and validation instructions.

## Signed Components

Production WinACME Framework v1.0.0 components are digitally signed using an Authenticode code-signing certificate issued to:

**Napolitano Security Consulting LLC**

Signed framework components include:

* WinACME Framework installer
* WinACME Framework executable
* Framework PowerShell modules
* Framework-maintained deployment scripts

Customer-provided deployment scripts are not part of the framework signing boundary unless they have been independently signed by an appropriate publisher.

Customers should verify the Authenticode signature and SHA-256 checksum of the downloaded installer before installation.

See [SECURITY.md](./SECURITY.md) for software verification guidance.

## Upgrade Protection

The v1.0.0 installer distinguishes between framework-maintained files and customer-provided deployment scripts.

Before replacing existing framework-maintained components, the installer creates a backup under:

```text
C:\ProgramData\WinACME-Framework\Backups
```

Customer-provided scripts that are not included in the framework distribution are preserved.

## Platform Requirements

Version 1.0.0 requires:

* Microsoft Windows Server
* Windows PowerShell 5.1
* Administrator privileges
* Appropriate permissions for the Windows Local Machine certificate store
* Appropriate administrative permissions for the selected deployment target
* Network connectivity to the configured ACME service
* simple-acme

Additional application-specific requirements may apply.

## Known Limitations

The following limitations apply to v1.0.0:

### Exchange Multi-Server Deployment

The included Exchange Hybrid deployment integration targets a single on-premises Exchange server.

Automated multi-server Exchange deployment is not included in this release.

### Exchange Relay Servers

Automated certificate deployment to separate Exchange relay servers is not included in v1.0.0.

### Remote Desktop Gateway Deployment Script

The Remote Desktop Gateway deployment script `ImportRDSFull.ps1` is not included with the framework distribution and must be supplied separately by the customer.

### Customer-Provided Scripts

WinACME Framework can execute customer-provided deployment scripts, but those scripts are outside the framework-maintained code and software-signing boundary.

Customers are responsible for reviewing, testing, securing, and maintaining their own deployment scripts.

### Application-Specific Environments

Application configurations can vary significantly between organizations.

Customers should validate certificate enrollment, deployment, application binding, and renewal behavior in an appropriate non-production environment before enabling automated production deployment.

## Production Validation

Before production use, customers should validate the complete certificate lifecycle for their intended deployment, including:

1. Framework installation
2. simple-acme installation and configuration
3. ACME service connectivity
4. Certificate enrollment
5. Certificate storage
6. Application-specific deployment
7. Application certificate binding
8. Renewal configuration
9. Certificate renewal
10. Post-renewal application deployment

Successful initial certificate issuance alone should not be considered sufficient validation of an automated certificate lifecycle.

The renewal and post-renewal deployment workflow should also be tested.

## Security

Customers should verify the authenticity and integrity of all production release packages before installation.

At minimum:

```powershell
Get-AuthenticodeSignature .\WinACME-Framework-Setup.exe
```

should report:

```text
Valid
```

and:

```powershell
Get-FileHash .\WinACME-Framework-Setup.exe -Algorithm SHA256
```

should match the SHA-256 checksum published with the release.

See [SECURITY.md](./SECURITY.md) for additional information.

## Documentation

Additional v1.0.0 documentation:

* [WinACME Framework Overview](./README.md)
* [Installation Guide](./INSTALLATION.md)
* [Security and Software Verification](./SECURITY.md)

## Support

WinACME Framework is developed and maintained as part of the PKIAdvisers certificate automation solutions portfolio.

Customers receiving the framework through a consulting or certificate automation engagement should use their established support channel for implementation and support requests.

## Publisher

**Napolitano Security Consulting LLC**

WinACME Framework is distributed through the **PKIAdvisers Product Solutions** repository.
