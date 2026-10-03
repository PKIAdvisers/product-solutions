# WinACME-Framework 1.0.0 Architecture

**Version documented:** 1.0.0  
**Platform:** Windows PowerShell 5.1  
**Primary runtime root:** `C:\ProgramData\WinACME-Framework`  
**Default simple-acme installation path:** `C:\Tools\simple-acme`

## 1. Purpose

WinACME-Framework 1.0.0 is an administrative orchestration layer around simple-acme. It provides a consistent workflow for installing simple-acme, defining a certificate profile, configuring certificate storage and renewal behavior, selecting deployment scripts, protecting ACME account secrets, provisioning certificates, validating selected deployment environments, running diagnostics, and removing simple-acme automation.

The framework does not implement the ACME protocol itself. Certificate enrollment and renewal are delegated to `wacs.exe`. The framework prepares configuration, validates prerequisites, builds the simple-acme command line, invokes simple-acme, and coordinates deployment-specific behavior.

## 2. Architectural Principles

Release 1.0.0 follows several design principles:

- **PowerShell 5.1 compatibility.** The framework is designed for Windows Server environments where Windows PowerShell 5.1 is the baseline.
- **Administrative execution.** The entry point requires elevation because installation, certificate-store operations, scheduled-task management, deployment, ACL changes, and other operations may require administrator rights.
- **Persistent operational state outside the application binary.** Runtime configuration, logs, secrets, backups, and deployment scripts are maintained beneath `C:\ProgramData\WinACME-Framework`.
- **Separation of secrets from ordinary configuration.** Sensitive values are stored separately and protected with Windows DPAPI rather than written directly to `config.json`.
- **Delegation to simple-acme.** The framework controls and validates the enrollment workflow but leaves ACME protocol operations and renewal state to simple-acme.
- **Pluggable deployment.** Deployment behavior is selected independently from certificate enrollment and may use built-in or custom deployment scripts.
- **Validation before execution.** Provisioning and deployment-specific checks are performed before the framework launches certificate issuance where applicable.
- **Operational observability.** Logging and diagnostics provide visibility into framework state, simple-acme state, certificate configuration, deployment configuration, and automation.

## 3. High-Level Architecture

```text
┌─────────────────────────────────────────────────────────────────────┐
│                     WinACME-Framework 1.0.0                         │
│                                                                     │
│  Start-WinACME-Framework.ps1                                       │
│          │                                                          │
│          ├── Elevation / environment validation                     │
│          ├── Module loading                                         │
│          ├── Application initialization                             │
│          └── Interactive menu                                       │
│                    │                                                │
│        ┌───────────┼──────────────┬───────────────┐                 │
│        │           │              │               │                 │
│   Configuration  Installer    Provisioning    Diagnostics           │
│        │           │              │               │                 │
│      Wizard     simple-acme       Deployment      Automation           │
│        │                          │                                  │
│      Secrets                 Exchange Hybrid                        │
│                                   │                                 │
│                              Custom Scripts                         │
│                              (for example SSRS)                     │
└───────────────────────────────┬─────────────────────────────────────┘
                                │
                                ▼
                    ┌───────────────────────┐
                    │       simple-acme        │
                    │       wacs.exe        │
                    └───────────┬───────────┘
                                │
                     ACME protocol / EAB
                                │
                                ▼
                    ┌───────────────────────┐
                    │       ACME CA         │
                    └───────────────────────┘
```

The framework is therefore best understood as an **orchestration and policy layer**. simple-acme remains the ACME client and owns the underlying ACME enrollment/renewal mechanics.

## 4. Entry Point and Runtime Bootstrap

`Start-WinACME-Framework.ps1` is the application entry point.

Startup proceeds in the following order:

```text
Start-WinACME-Framework.ps1
          │
          ▼
Validate PowerShell environment
          │
          ▼
Require administrative privileges
          │
          ├── Script execution → relaunch PowerShell with RunAs
          │
          └── Packaged EXE → elevation expected from package manifest
          │
          ▼
Import framework modules
          │
          ▼
Validate required commands
          │
          ▼
Initialize application directories/configuration
          │
          ▼
Initialize protected secret storage
          │
          ▼
Load application configuration
          │
          ▼
Initialize operational logging
          │
          ▼
Enter main menu loop
```

### 4.1 Module Import Order

The entry point imports the core modules in this order:

1. `Logging.psm1`
2. `Configuration.psm1`
3. `Secrets.psm1`
4. `Menu.psm1`
5. `Installer.psm1`
6. `Wizard.psm1`
7. `Deployment.psm1`
8. `ExchangeHybrid.psm1`
9. `Provisioning.psm1`
10. `Diagnostics.psm1`
11. `Automation.psm1`

`SSRS.psm1` is not imported by the main application bootstrap. It is a deployment-specific module used by the SSRS certificate deployment implementation rather than a core interactive framework dependency.

## 5. Runtime Directory Architecture

The framework maintains persistent operational data beneath:

```text
C:\ProgramData\WinACME-Framework
│
├── Config\
│   └── config.json
│
├── Logs\
│
├── Backups\
│
├── Secrets\
│
└── Deployments\
    ├── BuiltIn\
    └── Custom\
```

The default simple-acme application location is separate:

```text
C:\Tools\simple-acme
│
├── wacs.exe
├── settings.json
└── ...
```

This separation allows the framework's persistent configuration and operational data to remain independent of the simple-acme application installation.

## 6. Module Responsibilities

| Module | Architectural responsibility |
|---|---|
| `Logging.psm1` | Initializes framework logging and provides the common logging function. |
| `Configuration.psm1` | Creates the runtime directory structure, initializes/defaults the application configuration, loads/saves `config.json`, and exposes configuration state. |
| `Secrets.psm1` | Maintains protected machine-scoped secrets and applies restrictive filesystem ACLs. |
| `Menu.psm1` | Presents the interactive application menu and user-pause behavior. |
| `Installer.psm1` | Selects, validates, backs up, and installs/upgrades simple-acme from a ZIP package. |
| `Wizard.psm1` | Collects and validates certificate profile, storage, renewal, and Exchange Hybrid configuration. |
| `Deployment.psm1` | Discovers built-in/custom deployment scripts and persists the selected deployment. |
| `ExchangeHybrid.psm1` | Performs Exchange Hybrid environment discovery, prerequisite checks, baseline capture, and post-deployment validation. |
| `Provisioning.psm1` | Validates provisioning configuration, adjusts simple-acme settings, builds the command line, protects command display, launches `wacs.exe`, and coordinates provisioning. |
| `Diagnostics.psm1` | Evaluates framework, simple-acme, certificate profile, deployment, and automation health. |
| `Automation.psm1` | Discovers simple-acme renewals and scheduled tasks and provides controlled removal/cancellation workflows. |
| `SSRS.psm1` | Implements SSRS-specific certificate discovery, private-key access, SSL binding installation, validation, and rollback behavior. |

For the complete function-level API and internal helper inventory, see `MODULE-REFERENCE-1.0.0.md`.

## 7. Interactive Control Plane

The main menu is the administrative control plane for Release 1.0.0.

```text
Main Menu
   │
   ├── Install / Upgrade simple-acme
   │       └── Installer.psm1
   │
   ├── Configure Certificate Profile
   │       └── Wizard.psm1
   │
   ├── Configure Certificate Storage
   │       └── Wizard.psm1
   │
   ├── Configure Renewal Policy
   │       └── Wizard.psm1
   │
   ├── Select Deployment
   │       └── Deployment.psm1
   │
   ├── Configure Exchange Hybrid
   │       └── Wizard.psm1 + ExchangeHybrid.psm1
   │
   ├── Show Configuration
   │       └── Configuration.psm1
   │
   ├── Preview simple-acme Command
   │       └── Provisioning.psm1
   │
   ├── Provision Certificate
   │       └── Provisioning.psm1
   │
   ├── Diagnostics
   │       └── Diagnostics.psm1
   │
   ├── Remove Automation
   │       └── Automation.psm1
   │
   └── Exit
```

The menu itself contains little business logic. Its primary responsibility is routing operator selections to the appropriate module.

## 8. Configuration Architecture

`Configuration.psm1` owns the framework's persistent non-secret configuration.

The configuration object includes the application identity/version, framework paths, simple-acme installation metadata, certificate profile, storage configuration, deployment configuration, and renewal configuration.

Conceptually:

```text
config.json
│
├── Application
│
├── Paths
│
├── WinAcme
│
├── CertificateProfile
│
├── Storage
│
├── Deployment
└── Renewal
```

Configuration changes are persisted through `Save-ApplicationConfiguration`. The module writes to a temporary file, validates that the generated JSON can be parsed, and then moves the temporary file into place. This reduces the chance of leaving a partially written configuration file.

The configuration module also performs schema/default maintenance during initialization so that expected properties exist when the framework loads an existing configuration.

## 9. Secret Storage Architecture

Sensitive ACME account material is separated from `config.json`.

```text
Framework / Wizard / Provisioning
              │
              ▼
        Secrets.psm1
              │
      plaintext only in memory
              │
              ▼
       Windows DPAPI Protect
        Machine scope + entropy
              │
              ▼
C:\ProgramData\WinACME-Framework\Secrets
              │
              └── restrictive NTFS ACLs
```

`Secrets.psm1` uses `.NET` `ProtectedData.Protect()` and `ProtectedData.Unprotect()` with `DataProtectionScope.LocalMachine`. It also applies ACL protection to the secrets directory and individual secret files.

This architecture allows unattended machine-level automation to retrieve required protected values while avoiding storage of the secret as ordinary plaintext configuration.

## 10. Certificate Provisioning Architecture

`Provisioning.psm1` is the primary bridge between framework configuration and simple-acme.

### 10.1 Provisioning Flow

```text
Operator selects "Provision Certificate"
                  │
                  ▼
        Load framework configuration
                  │
                  ▼
      Validate provisioning configuration
                  │
          ┌───────┴────────┐
          │                │
        invalid           valid
          │                │
          ▼                ▼
 Display errors      Domain authorization notice
                           │
                           ▼
                Deployment/environment checks
                           │
                           ▼
                 Locate simple-acme settings
                           │
                           ├── Set renewal policy
                           └── Set key-exportability policy
                           │
                           ▼
                 Resolve wacs.exe
                           │
                           ▼
              Build provisioning arguments
                    ┌──────┼───────┐
                    │      │       │
              Certificate Storage Deployment
                 profile    args     args
                    └──────┼───────┘
                           │
                           ▼
                Produce sanitized preview
                           │
                           ▼
                     Invoke wacs.exe
                           │
                           ▼
                  Capture exit status
                           │
                           ▼
               Deployment/post-validation
```

### 10.2 Command Construction

Provisioning is intentionally decomposed into separate concerns:

- certificate/profile arguments;
- storage arguments;
- deployment arguments;
- ACME/EAB-related values;
- simple-acme settings changes;
- sanitized command rendering;
- process execution.

This separation allows the framework to display a safe preview of the command without unnecessarily exposing sensitive material.

### 10.3 simple-acme Settings

Before provisioning, the framework can update simple-acme configuration such as:

- `ScheduledTask.RenewalDays`;
- private-key exportability behavior required by the selected storage/deployment scenario.

The settings file is resolved from `settings.json`, with `settings_default.json` used as the available fallback source when appropriate.

## 11. ACME Boundary

The framework does not directly perform ACME account/order/challenge/finalization operations.

The architectural boundary is:

```text
WinACME-Framework
       │
       │ configuration + command-line arguments
       ▼
    wacs.exe
       │
       │ ACME protocol over HTTPS
       ▼
    ACME CA
```

For the Release 1.0.0 use case, certificate enrollment is designed around organization-controlled/pre-authorized certificate profiles and External Account Binding (EAB) rather than the framework implementing HTTP-01 or DNS-01 challenge handlers.

simple-acme remains responsible for its own renewal records and scheduled renewal execution.

## 12. Deployment Architecture

Certificate deployment is separated from enrollment.

```text
Certificate provisioning
          │
          ▼
 Deployment configuration
          │
    ┌─────┴─────┐
    │           │
 Built-in     Custom
    │           │
    └─────┬─────┘
          ▼
 Selected PowerShell deployment script
          │
          ▼
 Target application/service
```

`Deployment.psm1` discovers scripts beneath the framework deployment directories, records the selected deployment in configuration, and makes that selection available to provisioning.

`Provisioning.psm1` converts the selected deployment into the corresponding simple-acme deployment arguments.

This creates an important separation:

**certificate lifecycle management** is handled by simple-acme, while **application-specific certificate installation** can be implemented independently in deployment scripts.

## 13. Exchange Hybrid Deployment Pattern

Exchange Hybrid receives additional first-class validation in Release 1.0.0.

```text
Exchange Hybrid configuration
          │
          ▼
 Discover target Exchange server
          │
          ▼
 Establish PowerShell session
          │
          ▼
 Read current Exchange state
          │
          ▼
 Validate prerequisites
          │
          ▼
 Capture/log baseline
          │
          ▼
 Certificate provisioning/deployment
          │
          ▼
 Capture new certificate state
          │
          ▼
 Post-deployment validation
          │
          ├── Certificate
          ├── Send connector
          └── Receive connector
          │
          ▼
 Report success / validation failure
```

The architectural significance is that deployment is not treated merely as "run a script." The framework can perform environment-specific preflight and post-deployment validation around the certificate operation.

## 14. SSRS Deployment Pattern

SSRS demonstrates the custom deployment model.

`SSRS.psm1` is deployment-specific and is not part of the normal framework bootstrap. Its responsibilities include:

```text
Discover SSRS WMI namespace/configuration
                 │
                 ▼
        Read existing SSL bindings
                 │
                 ▼
       Validate candidate certificate
                 │
                 ▼
      Resolve certificate private key
                 │
                 ▼
 Grant SSRS service account read access
                 │
                 ▼
       Remove/replace SSL bindings
                 │
                 ▼
          Validate new bindings
                 │
            failure?
           /       \
         no         yes
         │           │
         ▼           ▼
      Complete    Attempt rollback
```

The rollback behavior is important: the deployment implementation retains information about original bindings and attempts to restore them when installation fails.

This pattern illustrates how Release 1.0.0 can extend certificate lifecycle automation to applications that require application-specific certificate binding logic.

## 15. Renewal and Automation Architecture

The framework does not replace simple-acme's renewal engine.

After successful provisioning:

```text
Initial framework provisioning
           │
           ▼
        simple-acme
           │
           ├── renewal configuration/state
           │
           └── Windows Scheduled Task
                       │
                       ▼
                 Future execution
                       │
                       ▼
                    simple-acme
                       │
                       ▼
                ACME certificate renewal
                       │
                       ▼
                 Deployment script
```

`Automation.psm1` is primarily an administrative and cleanup layer around this automation. It discovers renewal files and scheduled tasks and provides controlled cancellation/removal operations.

This distinction is important:

> **The framework configures and manages the automation; simple-acme performs recurring certificate renewal.**

## 16. Automation Removal Flow

Automation removal is deliberately guarded because deleting renewal state or scheduled tasks can disable future certificate renewal.

```text
Remove Automation
       │
       ▼
Display safety notice
       │
       ▼
Discover renewal records
       │
       ├── Remove selected renewal
       ├── Remove all renewals
       └── Leave renewals unchanged
       │
       ▼
Re-evaluate remaining renewal state
       │
       ▼
Remove scheduled tasks only when appropriate
       │
       ▼
Recommend diagnostics
```

The module uses simple-acme cancellation where appropriate rather than simply deleting arbitrary state files.

## 17. Diagnostics Architecture

Diagnostics aggregate checks across the major framework layers:

```text
Show-FrameworkDiagnostics
           │
           ▼
 Get-FrameworkDiagnostics
           │
   ┌───────┼────────────────┐
   │       │                │
Framework simple-acme    Certificate Profile
   │       │                │
   ├──── Deployment ────────┤
   │                        │
   └──── Automation ────────┘
           │
           ▼
   Diagnostic result objects
           │
           ├── PASS
           ├── WARNING
           └── FAIL
           │
           ▼
 Console summary + diagnostic logging
```

The resulting overall status is reported as:

- `HEALTHY`
- `HEALTHY WITH WARNINGS`
- `UNHEALTHY`

This provides a common health model across otherwise independent modules.

## 18. Logging Architecture

`Logging.psm1` is imported first because logging is a cross-cutting framework service.

```text
Framework modules
      │
      ▼
   Write-Log
      │
      ▼
C:\ProgramData\WinACME-Framework\Logs
```

Logging is used for application lifecycle events, configuration changes, secret-management events, installation activity, provisioning, validation, and error reporting.

The entry point also attempts to prevent a logging failure from unnecessarily breaking interactive error handling.

## 19. Trust and Privilege Boundaries

Release 1.0.0 crosses several important security boundaries.

### 19.1 Administrator Boundary

The framework requires administrative privileges. Operations may affect:

- `C:\ProgramData`;
- the Windows certificate store;
- private-key ACLs;
- Windows Scheduled Tasks;
- application/service certificate bindings;
- the simple-acme installation;
- remote Exchange configuration.

### 19.2 Secret Boundary

Sensitive values are protected through machine-scoped DPAPI and restrictive ACLs. Ordinary configuration remains separate from secret material.

### 19.3 simple-acme Boundary

The framework launches an external ACME client. simple-acme is responsible for ACME transactions, renewal-state formats, and scheduled renewal behavior.

### 19.4 Deployment Boundary

Deployment scripts may modify external services. Their privileges and impact therefore extend beyond the framework's own filesystem.

### 19.5 Remote Management Boundary

Exchange Hybrid validation may establish a remote PowerShell session and inspect or modify Exchange-related configuration through the deployment workflow.

## 20. Component Dependency View

```text
                         ┌──────────────┐
                         │   Logging    │
                         └──────▲───────┘
                                │
                    used across framework
                                │
┌──────────────┐        ┌───────┴────────┐
│   Secrets    │◄───────│ Configuration  │
└──────▲───────┘        └───────▲────────┘
       │                        │
       │                 ┌──────┴──────┐
       │                 │   Wizard    │
       │                 └──────▲──────┘
       │                        │
       │                 ┌──────┴──────┐
       └─────────────────│ Provisioning│
                         └───▲──────▲───┘
                             │      │
                   ┌─────────┘      └──────────┐
                   │                           │
            ┌──────┴──────┐            ┌──────┴─────────┐
            │ Deployment  │            │ ExchangeHybrid │
            └──────┬──────┘            └────────────────┘
                   │
             deployment scripts
                   │
          ┌────────┴─────────┐
          │                  │
    Built-in scripts     Custom scripts
                              │
                              └── SSRS pattern

      ┌─────────────┐                ┌─────────────┐
      │ Diagnostics │                │ Automation  │
      └──────▲──────┘                └──────▲──────┘
             │                              │
             └──── inspect framework ───────┘
                    and simple-acme state
```

The diagram is conceptual rather than a strict PowerShell import graph. It shows responsibility and operational dependencies.

## 21. External Dependencies

The architecture depends on several Windows and external components:

| Dependency | Purpose |
|---|---|
| Windows PowerShell 5.1 | Framework runtime |
| Administrator token/UAC | Required privileged operations |
| simple-acme / `wacs.exe` | ACME client, certificate issuance, renewal state, scheduled renewal |
| Windows DPAPI | Machine-scoped secret protection |
| NTFS ACLs | Filesystem protection for secret material |
| Windows Certificate Store | Certificate storage for applicable profiles |
| Windows Task Scheduler | Recurring simple-acme renewal execution |
| PowerShell remoting | Exchange Hybrid discovery/validation where configured |
| Application-specific APIs/WMI | Deployment targets such as SSRS |

## 22. Release 1.0.0 Architectural Constraints

Several characteristics of 1.0.0 are intentional baseline constraints and are important when comparing the release with later versions:

1. **One primary framework configuration is maintained per installation.**
2. **Certificate profile, storage, deployment, and renewal settings are represented in the central application configuration.**
3. **Deployment selection is configuration-driven but does not yet use a generic parameterized deployment manifest.**
4. **Provider/account configuration is not yet modeled as reusable provider profiles.**
5. **The framework does not yet act as a multi-automation hub.**
6. **Application-specific integrations can require specialized module/script logic.**
7. **simple-acme remains the recurring renewal engine rather than the framework implementing its own renewal scheduler.**

These constraints provide the architectural baseline against which Release 2.0.0 can be documented.

## 23. Architectural Summary

The Release 1.0.0 architecture can be reduced to five layers:

```text
┌───────────────────────────────────────────────────────────┐
│  1. Operator / Control Layer                              │
│     Start script, Menu, Wizard, Diagnostics               │
├───────────────────────────────────────────────────────────┤
│  2. Framework State & Security Layer                      │
│     Configuration, Secrets, Logging                       │
├───────────────────────────────────────────────────────────┤
│  3. Certificate Orchestration Layer                       │
│     Provisioning, Deployment, Automation                  │
├───────────────────────────────────────────────────────────┤
│  4. Application Integration Layer                         │
│     Exchange Hybrid, SSRS, custom deployment scripts      │
├───────────────────────────────────────────────────────────┤
│  5. External Execution Layer                              │
│     simple-acme, Windows certificate store, Task Scheduler,  │
│     ACME CA, target applications/services                 │
└───────────────────────────────────────────────────────────┘
```

The central architectural idea is that **WinACME-Framework 1.0.0 coordinates certificate lifecycle operations without replacing the underlying ACME client or the target application's certificate-management mechanisms**.

That separation is what allows the framework to provide common configuration, security, validation, diagnostics, and operational controls while still supporting application-specific deployment logic.
