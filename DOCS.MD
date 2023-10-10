# Terraform Manager
Terraform Manager extends the Terraform CLI with additional functonalities and aids to standardize Terraform code structure throughout the entire development lifecycle to enable more consistent Ops deployment of Terraform for use cases such as GitOps.

Terraform Manager consists of 3 parts:
1. Utils: Core Features of Terraform Manager
2. Terraformx: Extends Terraform Manager core features into a CLI
3. Actions: Turns basic terraformx commands into executables for use in custom CI actions.




# Table of Contents
- [Installation](#installation)
  - [MacOS](#macos)
  - [Windows](#windows)
- [Prerequisites](#prerequisites)
  - [Directory Layout](#directory-layout)
  - [Workflow File Example](#workflow-file-example)
- [Features](#features)
  - [Terraform Manager](#terraform-manager-1)
  - [Terraformx](#terraformx)
  - [Terraformx Actions](#terraformx-actions)
- [List of Related Repositories](#list-of-related-repositiories)
- [Workflows in this Repository](#workflows-in-this-repository)



# Installation
This section includes instructions on how to install Terraform Manager and Terraformx.
## MacOS
### Installation of Terraform Manager
In the root directory of terraform_manager, use the following shell scripts to install the exe and update the PATH and alias:

- `one-dir-terraform-manager-install.sh`
- `one-file-terraform-manager-install.sh`

### Installation of Terraformx
#### Manual Installation
In the root directory of terraformx, use the following shell scripts to install the exe and update the PATH and alias:

  - `one-dir-terraformx-install.sh`
  - `one-file-terraformx-install.sh`

#### Install with Homebrew
- `brew tap jlieow/terraform-manager`
- `brew install terraformx`



## Windows
### Installation of Terraform Manager
In the root directory of terraform_manager, run:
- `pyinstaller --onefile main.py -p %cd% -y`

After PyInstaller has completed follow the following instructions to set the alias:
1. Create a .bat or .cmd file with your DOSKEY commands.
- ```
    @echo off

    DOSKEY terraform_manager=C:\path_to_exe\main.exe
    ```

1. Run regedit and go to HKEY_LOCAL_MACHINE\Software\Microsoft\Command Processor

2. Add String Value entry with the name AutoRun and the full path of your .bat/.cmd file.

### Installation of Terraformx
#### Manual Installation
In the root directory of terraformx, run:
- `pyinstaller --onefile terraformx.py -p %cd% -y`


Follow the instructions for installing Terraform Manager above but replace the DOSKEY command with:
- `DOSKEY terraformx=C:\path_to_exe\terraformx.exe`

#### Install with Scoop
- `scoop bucket add <name> https://github.com/jlieow/terraform_manager_bucket`
- `scoop install terraformx`




# Prerequisites
Before you start, read this section to get an understanding of what your directories and workflow file (if you choose to use workflows) should look like.
## Directory Layout
The diagram below shows what a typical directory looks like. It includes some files and folders introduced and used by the tool.

    .
    ├── config                # Terraform Manager and Terraformx CLI locates all your tfvars files in the root directory and combines them into a settings.tfvars file here.
    │   └── .env              # Environmental variables declared in a .env file will be loaded
    │   └── settings.tfvars   # Values in backend.tfvars and config.tfvars will be stored here
    ├── modules               # Location of Terraform modules.
    ├── workflow
    │   └── config.yaml       # Terraform Manager and Terraformx CLI will detect the yaml file and invoke terraform commands on targeted modules and resources.
    ├── backend.tf            # Tells Terraform where to store the Terraform state file.
    ├── main.tf               # Terraform main.tf.
    ├── output.tf             # Outputs values after provisioning resources.
    ├── variables.tf
    ├── backend.tfvars        # Variables to be used by backend.tf. This will be picked up by Terraform Manager and Terraformx CLI and placed into /config
    ├── config.tfvars         # Variables to be used by Terraform when provisioning your resources. This will be picked up by Terraform Manager and Terraformx CLI and placed into /config
    └── README.md



## Workflow File Example
Workflows is a method to perform terraform apply on targeted modules/resources by storing them as steps in a yaml file. By simply creating a directory called workflow on the root dir of a Terraform root and placing a config.yaml file in your terraform root, Terraform Manager and Terraformx CLI will detect the yaml file and invoke terraform commands on target modules and resources.

```
ignore: True
active_stages: 2-3, 5
active_stage_names: stage 1, stage 2
stages:
  - name: stage 1
    auto_approve: True
    targets:
      - module:
          - mi-1
          - mi-2
      - resource:
          - ri-1.extra
          - ri-2.extra
          - ri-1.try
  - name: stage 2
    auto_approve: True
    targets:
      - module:
          - mi-24
```

config.yaml legend:
| key | description | type | Example allowed values | Default value when omitted | Behaviour if omitted |
| --- | --- | --- | --- | --- | --- |
| ignore | True tells the Terraform Manager or Terraformx CLI to ignore the entire workflow file | bool | True, False | False | config.yaml is not ignored |
| active_stages | Specifies which stage should be applied or destroyed and ignores the rest using an int | int | 1, 3-4 | None | No stage is ignored |
| active_stage_names | Specifies which stage should be applied or destroyed and ignores the rest using the name of the stage | int | stage 1, stage 2 | None | No stage is ignored |
| stages | Specifies the stages | arr | [] | None | Operation would fail |
| stages.name | Name of the stage | str | "stage 1" | None | Stage would not have a name but will continue to run |
| stages.auto_approve | Evaluated to see if the stage would should be auto approved without user input | bool | True, False | False | Stage will not be auto approved and require user input |
| stages.targets | Specifies which modules or resources should be built in this stage | dict | module, resource | None | Operation would fail |
| stages.targets.module | Specifies the name of the modules which should be built in this stage | arr | "name_of_module" | None | No modules will be targeted |
| stages.targets.resources | Specifies the name of the resources which should be built in this stage | arr | "type_of_resource.name_of_resource" | None | No resources will be targeted |




# Features
## Terraform Manager
1. Import AWS profiles from CSVs stored in /aws_credentials
    
    - Import .csv profiles found in /aws_credentials using aws configure import --csv command.

2. Perform Terraform Commands

    - Detects if a workflow file exists in the terraform root and perform regular or more complex terraform commands

3. Check Terraform Roots for Configuration Drift

    - Detects configuration drift based on workflow configuration

4. Perform Terraform Multi-Build

    - Allows sequential building of multiple terrform roots

5. Apply Terraform Blueprints

    - Allows you to save sequential build steps into a csv file and reference them later on when a build is required.

6. Terraform Destroy based on History

    - Each time a terraform root is built, it is saved to a history .csv file. When required, the file can be later referenced to destroy each root sequentially.

7. Use Workflows

    - Workflows is a method to perform terraform apply on targeted modules/resources by storing them as steps in a yaml file.



## Terraformx
Commands:

1. terraformx init<br/><br/>

    The terraformx init command initializes a terraform root.<br/><br/>

    Optional Parameters:
    
    1. -ch-dir
        - Location of terraform root.
    2. -var-file        
        - Location of variable definitions file.
    <br/>

2. terraformx apply<br/><br/>

    The terraformx apply command executes the actions proposed in a Terraform plan.<br/><br/>

    Optional Parameters:
    
    1. -ch-dir
        - Location of terraform root.
    2. -var-file
        - Location of variable definitions file.
    3. -auto-approve
        - Auto approve command without requiring user input.
    4. -override-workflow
        - Overrides workflow stages auto_approve keys and auto approves every stage.
    5. -refresh-only
        - Review how terraform would update your state file.
    6. -rebuild
        - Rebuild terraform by destroying and applying the script.
    7. -blueprint
        - Location of blueprint file. Full and relative paths are allowed. If used without the parameter -create, the blueprint file is applied. If used with the parameter -create, a blueprint file is created.
    8. -create
        - Creates a blueprint. Must be used with the parameter -blueprint.
    <br/>

3. terraformx destroy<br/><br/>

    The terraformx destroy command destroys the resources managed by the terraform root.<br/><br/>

    Optional Parameters:
    
    1. -ch-dir
        - Location of terraform root.
    2. -var-file
        - Location of variable definitions file.
    3. -auto-approve
        - Auto approve command without requiring user input.
    4. -override-workflow
        - Overrides workflow stages auto_approve keys and auto approves every stage.
    5. -refresh-only
        - Review how terraform would update your state file.
    6. -destroy-history
        - Destroys all terraform roots' managed resources in terraform_history.csv.
    <br/>

4. terraformx output<br/><br/>

    The terraformx output command is used to extract the value of output variables from the terraform root.<br/><br/>

    Optional Parameters:
    
    1. -ch-dir
        - Location of terraform root.
    <br/>

5. terraformx import<br/><br/>

    The terraformx import command is used to import existing infrastructure into the state.<br/><br/>

    Optional Parameters:
    
    1. -ch-dir
        - Location of terraform root.
    2. -var-file
        - Location of variable definitions file.
    <br/>

6. terraformx state<br/><br/>

    The terraformx state command is used to manage the state.<br/><br/>

    Optional Parameters:
    
    1. -ch-dir
        - Location of terraform root.
    <br/>





## Terraformx Actions

The code transforms the following features into exes for custom GitHub Actions:

1. Terraformx Apply
2. Terraformx Destroy
3. Terraformx Rebuild

Please refer to the earlier sections for their behaviours.




# List of Related Repositiories 
### Terraformx Actions Repos
#### [Terraformx Apply](https://github.com/jlieow/action_terraformx_apply) - Contains a composite action that uses Terraformx Apply
#### [Terraformx Destroy](https://github.com/jlieow/action_terraformx_destroy) - Contains a composite action that uses Terraformx Destroy
#### [Terraformx Rebuild](https://github.com/jlieow/action_terraformx_rebuild) - Contains a composite action that uses Terraformx Rebuild


### Terraformx Installation
#### [Homebrew Tap](https://github.com/jlieow/homebrew-terraform-manager) - Homebrew tap containing the formula for Terraformx (MacOS install)
#### [Scoop Bucket](https://github.com/jlieow/terraform_manager_bucket) - Scoop Bucket containing the manifest for Terraformx (Windows install)




# Workflows in this Repository
### generate_linux_exe_terraformx_apply.yml
Runs PyInstaller to transform Terraformx Apply into an exe then creates a pull request in the Terraformx Apply repository to update the exe file.
### generate_linux_exe_terraformx_destroy.yml
Runs PyInstaller to transform Terraformx Destroy into an exe then creates a pull request in the Terraformx Destroy repository to update the exe file.
### generate_linux_exe_terraformx_rebuild_with_destroy_apply.yml
Runs PyInstaller to transform Terraformx Rebuild into an exe then creates a pull request in the Terraformx Rebuild repository to update the exe file.
### generate_terraformx_exe_mac.yml
Runs PyInstaller to transform the Terraformx CLI tool into an exe for MacOS then uploads it as an artifact
### generate_terraformx_exe_ubuntu.yml
Runs PyInstaller to transform the Terraformx CLI tool into an exe for Ubuntu then uploads it as an artifact
### generate_terraformx_exe_windows.yml
Runs PyInstaller to transform the Terraformx CLI tool into an exe for Windows then uploads it as an artifact
### update_homebrew_formula.yml
When a new Mac release is created in this repository, the Homebrew formula in the Homebrew tap is updated with the url to the new version along with the hash and a pull request is created. 
### update_homebrew_formula.yml
When a new Windows release is created in this repository, the Scoop manifest in the Scoop bucket is updated with the url to the new version along with the hash and a pull request is created. 