# Overview
Feature Update Controller is designed to streamline Windows upgrades for devices managed through Microsoft Intune. It provides centralized control over setup configurations, custom actions and script modules, ensuring a seamless and customizable upgrade experience.

![image](https://github.com/user-attachments/assets/47d85500-2789-4799-9f34-00d243008656)

# What does it do

- Create and prepare SetupConfig.ini for the Windows setup engine
- Prestage what's called 'Script Modules', essentially individual scripts with a specific purpose (such as configuring a default wallpaper)
- Prestage and configure the device to make use of what's known as 'Custom Actions' (read more about them [here](https://learn.microsoft.com/en-us/windows-hardware/manufacture/desktop/windows-setup-enable-custom-actions))

# How does it work
Feature Update Controller is a Remediation script package designed to provide control over the Windows Feature Update deployment process for devices managed by Microsoft Intune. Upgrading between Windows releases with Microsoft Intune today provides some capabilities, but lacks a way to control the setup parameters, prestage script files to control what happens during different stages of a Windows upgrade. For instance, the Windows setup utility uses default parameters and values, unless the following file exist locally on the device being upgraded: 'C:\Users\Default\AppData\Local\Microsoft\Windows\WSUS\SetupConfig.ini'. With the Feature Update Controller solution, you are provided with capabilities to not only create this file if desired, but also provide its configuration values, such as the example below:

```ini
[SetupConfig]
Priority=Normal
Compat=IgnoreWarning
DynamicUpdate=Enable
ShowOOBE=None
Telemetry=Enable
Uninstall=Enable
POSTOOBE=C:\ProgramData\<company_name>\FeatureUpdateController\SetupComplete.cmd
PostRollback=C:\ProgramData\<company_name>\FeatureUpdateController\SetupRollback.cmd
```

As shown in the above example, several properties and their respective values are provided within the SetupConfig.ini file. Upon a Windows upgrade event, the setup engine adheres to what's configured here, providing you as the administrator control over the experience. With the Feature Update Controller, all you need to configure centrally, is the desired properties and their respective values, in what's called the 'manifest' file (read more about it's configuration options in the section named 'Manifest configuration (manifest.json)' below). As for the creation of the SetupConfig.ini file in the appropriate location, but also updating if configuration changes in the manifest file, it's all taken care of by the Feature Update Controller.

From an overview perspective, these are the capabilities that the Feature Update Controller solution provides:
- Creation of SetupConfig.ini file in its required location
  - If POSTOOBE property is specified:
    - Create the following file: C:\ProgramData\Ericsson\FeatureUpdateController\SetupComplete.cmd
    - Download the SetupComplete.ps1 script from the specified Azure Storage Account
    - Modify SetupComplete.cmd to execute the SetupComplete.ps1 PowerShell script
  - If PostRollback property is specified:
    - Create the following file: C:\ProgramData\Ericsson\FeatureUpdateController\SetupRollback.cmd
    - Download the SetupRollback.ps1 script from the specified Azure Storage Account
    - Modify SetupRollback.cmd to execute the SetupRollback.ps1 PowerShell script
- Staging of Script Modules (separate scripts with a specific purpose)

# Prerequisites
Feature Update Controller requires a bit of preconfiguration before it can be enabled in an environment.

- PowerShell 5.0 or higher (on your devices)
- Azure Storage Account with appropriate public permissions
- Access to modify and deploy Remediations

# Configuration

Below are the different aspects of the solution that needs to be preconfigured before you start.

## Storage Account and Container

Before administrators deploy the Remediation script package to endpoints, a Storage Account resource in Azure must be created (or reuse an existing one) with a public container. Structure wise, the content (blobs) within the container is considered 'flat' from the Remediation script package perspective. This means that there should be no folder structure or similar. Here's an example:

- StorageAccountName/ContainerName/manifest.json
- StorageAccountName/ContainerName/Detection.ps1
- StorageAccountName/ContainerName/SetupComplete.ps1
- StorageAccountName/ContainerName/start2.bin
- StorageAccountName/ContainerName/...

Basically, everything that's referenced within the manifest file (manifest.json) in terms of script files or support files, should simply be added in the root of the container.

## Manifest configuration (manifest.json)

TBA

## Remediation script package file (Detection.ps1)

Within the Detection.ps1 script file, modify the following variables with values suitable for your organization:

```powershell
$CompanyName = "<company_name>"
$StorageAccountName = "<storage_account_name>"
$StorageAccountContainer = "<storage_account_container_name>"
```

**$CompanyName** 
  - Your organization's name
  - Example: "MSEndpointMgr"
  - Used for logging and directory path purposes
  - Don't use special characters that's not allowed for directory names

**$StorageAccountName**
  - The name of your Azure Storage account
  - Example: "az-mse-sa-fuc-store"
  - Must adhere to the rules of allowed characters for storage accounts in Azure

**$StorageAccountContainerName**
  - The name of the container within your Azure Storage account
  - Example: "data-prod"
  - Must be lowercase and can include hyphens

## Script Modules

TBA (explain each script module's purpose and their respective configuration requirements)

# Setup Instructions
1. Replace all placeholders (enclosed in `<>`) with your actual values in the following files:
   1. Detection.ps1
   2. SetupComplete.ps1
   3. SetupRollback.ps1
   4. Script Module files if used
2. Ensure that the required files are added to the Storage Account container specified within the Detection.ps1 script file
   1. Script Module files and support files
3. Configure manifest.json with desired settings 
4. Test the script in a controlled environment
5. Deploy to devices


## Notes
- Keep the manifest file name as "manifest.json" unless you have a specific reason to change it
- Ensure all Azure Storage account details are correct before deployment
- Test the script in a controlled environment before wide deployment

## Support
No support is provided for this solution. For issues and questions, please refer to the [Feature Update Controller GitHub repository](https://github.com/MSEndpointMgr/FeatureUpdateController).