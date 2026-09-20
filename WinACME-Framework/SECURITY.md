# WinACME Framework Security

This document describes the security and software verification practices for WinACME Framework releases distributed through the PKIAdvisers Product Solutions repository.

Customers should verify the authenticity and integrity of WinACME Framework software before installing or executing it in a production environment.

## Software Publisher

Production WinACME Framework components are digitally signed using an Authenticode code-signing certificate issued to:

**Napolitano Security Consulting LLC**

The digital signature allows customers to verify that a signed component was published by Napolitano Security Consulting LLC and has not been modified after signing.

Customers should independently verify digital signatures before executing downloaded software.

## Release Verification

Production releases may include:

```text
WinACME-Framework-Setup.exe
WinACME-Framework-Setup.exe.sha256
```

Customers should perform two independent checks before installation:

1. Verify the Windows Authenticode digital signature.
2. Verify the SHA-256 checksum against the value published with the release.

Both checks should succeed before the installer is executed.

## Verify the Installer Digital Signature

Open Windows PowerShell in the directory containing the downloaded installer and run:

```powershell
Get-AuthenticodeSignature .\WinACME-Framework-Setup.exe |
    Format-List Status, StatusMessage, SignerCertificate, TimeStamperCertificate
```

The expected signature status is:

```text
Status : Valid
```

Review the `SignerCertificate` information and verify that the certificate subject identifies:

```text
Napolitano Security Consulting LLC
```

Also review the `TimeStamperCertificate` information when present.

Do not install the software if:

* The signature status is not `Valid`.
* The expected publisher cannot be verified.
* Windows reports that the file has been modified after signing.
* The signature cannot be validated using the organization's normal certificate validation process.

## Verify the SHA-256 Checksum

Calculate the SHA-256 hash of the downloaded installer:

```powershell
Get-FileHash .\WinACME-Framework-Setup.exe -Algorithm SHA256
```

Example output:

```text
Algorithm : SHA256
Hash      : <SHA-256 HASH>
Path      : C:\Path\To\WinACME-Framework-Setup.exe
```

Compare the calculated hash with the SHA-256 value published with the corresponding release.

The values must match exactly.

A valid digital signature verifies the signed software publisher and integrity of the signed file. The separately published SHA-256 checksum provides an additional mechanism for confirming that the downloaded file matches the intended release artifact.

## Verify the Installed Framework Executable

After installation, verify the signature of the installed framework executable:

```powershell
Get-AuthenticodeSignature 'C:\ProgramData\WinACME-Framework\WinACME-Framework.exe' |
    Format-List Status, StatusMessage, SignerCertificate, TimeStamperCertificate
```

The expected status is:

```text
Valid
```

## Verify Framework Modules

Production framework modules are digitally signed.

Verify all installed modules with:

```powershell
Get-ChildItem 'C:\ProgramData\WinACME-Framework\Modules\*.psm1' |
    Get-AuthenticodeSignature |
    Select-Object Path, Status
```

Framework-distributed modules should report:

```text
Valid
```

Any unexpected signature status should be investigated before the framework is used for certificate management or deployment.

## Verify Deployment Scripts

Framework-maintained deployment scripts distributed with WinACME Framework are digitally signed.

They can be inspected using:

```powershell
Get-ChildItem 'C:\ProgramData\WinACME-Framework\Deployments\Custom\*.ps1' |
    ForEach-Object {
        $Signature = Get-AuthenticodeSignature $_.FullName

        [PSCustomObject]@{
            Script = $_.Name
            Status = $Signature.Status
            Signer = $Signature.SignerCertificate.Subject
        }
    }
```

For WinACME Framework v1.0.0, framework-distributed deployment scripts include:

```text
Install-ExchangeHybrid-Certificate.ps1
Install-SSRS-Certificate.ps1
```

These components should have a valid Authenticode signature identifying the expected publisher.

## Customer-Provided Deployment Scripts

WinACME Framework supports customer-provided PowerShell deployment scripts under:

```text
C:\ProgramData\WinACME-Framework\Deployments\Custom
```

Customer-provided scripts are outside the framework's software-signing trust boundary unless they have independently been signed by an appropriate publisher.

For example:

```text
ImportRDSFull.ps1
```

is customer-provided and is not distributed as a WinACME Framework v1.0.0 component.

A customer-provided script may therefore have a different Authenticode signing status from framework-maintained components.

Organizations are responsible for reviewing, approving, securing, and maintaining customer-provided deployment scripts according to their own security and change-management requirements.

Framework installation or upgrade operations are designed to preserve customer-provided deployment scripts that are not part of the framework distribution.

## PowerShell Execution Policy

WinACME Framework does not require customers to weaken organization-wide PowerShell security policies in order to establish trust in the software publisher.

Customers should not disable security controls or configure unrestricted PowerShell execution solely to run WinACME Framework.

PowerShell execution policy should remain consistent with organizational security policy.

Where an organization requires signed PowerShell code, framework-distributed PowerShell components can be validated using Windows Authenticode.

Customer-provided scripts should be handled according to the organization's own PowerShell signing and execution requirements.

## Publisher Trust

A valid Authenticode signature can be verified without automatically placing the publisher certificate into the Windows Trusted Publishers store.

Organizations that choose to explicitly trust **Napolitano Security Consulting LLC** as a software publisher should do so through their established endpoint-management, Group Policy, certificate-management, or other enterprise trust-management process.

The publisher certificate should not be added to the **Trusted Root Certification Authorities** store merely for the purpose of trusting signed WinACME Framework software.

Trust decisions remain under the control of the customer organization.

## Administrative Privileges

WinACME Framework performs operations that may require elevated Windows privileges, including:

* Accessing Local Machine certificate stores
* Installing or configuring certificate material
* Creating or managing certificate renewal automation
* Configuring application certificate bindings
* Executing application-specific deployment operations
* Writing framework configuration and operational data

The framework should therefore be executed only by authorized administrators.

Access to the server and the framework installation directory should be restricted according to organizational security policy.

## Sensitive Configuration

ACME environments may require sensitive configuration values such as:

* External Account Binding Key Identifiers
* EAB HMAC keys
* Other ACME service credentials or secrets

These values should be treated as sensitive authentication material.

Do not store sensitive values in:

* Public Git repositories
* Documentation
* Unsecured scripts
* Tickets or chat systems without appropriate protection
* General-purpose shared folders

Access to:

```text
C:\ProgramData\WinACME-Framework\Secrets
```

and other locations containing certificate automation credentials should be limited to authorized accounts.

## Private Keys

Certificates managed by WinACME Framework may include private keys stored in the Windows Local Machine certificate store.

Private keys should be protected according to organizational PKI policy.

Administrators should consider:

* Appropriate private-key permissions
* Whether private keys should be exportable
* Access granted to application service accounts
* Certificate backup requirements
* Protection of exported PFX or PKCS#12 files
* Secure removal of temporary certificate or key material

Private keys should never be committed to a source-code repository or placed in unsecured shared storage.

## simple-acme

WinACME Framework uses [simple-acme](https://simple-acme.com/) as its ACME client.

simple-acme is a separate project and is not developed or maintained by PKIAdvisers or Napolitano Security Consulting LLC.

Customers should obtain simple-acme from its official distribution source and follow the project's security and installation guidance.

The security and release practices of the simple-acme project are independent of the WinACME Framework software-signing and release process.

For additional information, visit the [official simple-acme website](https://simple-acme.com/).

## Release Integrity

Each production WinACME Framework release should be treated as a specific set of release artifacts.

Customers should use the installer associated with the intended release and verify its digital signature and published checksum before deployment.

Do not substitute an installer from an unknown or unofficial distribution source simply because the filename or version appears to match an official release.

## Production Deployment

Before enabling automated certificate renewal and deployment in a production environment:

1. Validate the installation package and signatures.
2. Test the certificate enrollment workflow.
3. Confirm the expected certificate is issued.
4. Test the application-specific deployment operation.
5. Verify the application is using the expected certificate.
6. Test the renewal process.
7. Confirm post-renewal deployment succeeds.
8. Review framework logs for unexpected errors.
9. Establish an operational process for monitoring certificate renewal.

Testing should be performed in an appropriate non-production environment before automated deployment is enabled on production systems.

## Reporting Security Issues

Potential security issues involving WinACME Framework should be reported privately through the customer's established PKIAdvisers or Napolitano Security Consulting LLC support channel.

Please do not publicly disclose suspected vulnerabilities before the issue has been reviewed and an appropriate remediation or disclosure process has been established.

Security reports should include, where possible:

* WinACME Framework version
* Windows Server version
* Affected component
* Description of the issue
* Steps required to reproduce the behavior
* Relevant framework logs
* Expected behavior
* Observed behavior

Do not include private keys, passwords, EAB HMAC keys, access tokens, or other sensitive credentials in security reports.

## Additional Documentation

* [WinACME Framework Overview](./README.md)
* [Installation Guide](./INSTALLATION.md)
* [Release Notes](./RELEASE-NOTES.md)

For simple-acme documentation and project information, visit the [official simple-acme website](https://simple-acme.com/).
