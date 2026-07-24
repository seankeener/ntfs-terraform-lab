# Watch me do it here:

https://loom.com/share/2051417c3fb3417cb089e0eb9777ba27

# Standard Operating Procedure
## Build and Validate a Terraform-Automated Active Directory Lab in Azure

| Field | Details |
|---|---|
| **Lab Title** | Terraform-Automated Active Directory Lab in Azure |
| **Tools Used** | Terraform · Azure CLI · PowerShell · Visual Studio Code |
| **Infrastructure** | 3 Azure VMs — domain controller, file server, client workstation |
| **Departments Configured** | HR · IT · Finance · Sales |
| **Users Validated** | Sarah · Lisa · HR user · John Smith · Tom Davis |

---

## Objective

This SOP explains how to provision a self-contained Active Directory lab in Azure using Terraform and PowerShell automation, then validate file share permissions and RDP access for multiple user roles. It is designed so a team member can reproduce the environment, run the automation, and confirm that access controls work as intended.

---

## Prerequisites

- Active Azure subscription with sufficient permissions and resource quota
- The following tools installed on a Windows workstation:
  - **Azure CLI for Windows**
  - **Terraform for Windows**
  - **Visual Studio Code**
- All tools confirmed accessible from the command line before beginning
- Lab folder containing Terraform files and PowerShell scripts open in VS Code

---

## Step 1 — Prepare the Required Tools

Install and verify all tools before running any Terraform or PowerShell commands.

- Install **Azure CLI for Windows** from the official Microsoft documentation
- Install **Terraform for Windows** from hashicorp.com
- Install **Visual Studio Code**
- Open a terminal and confirm each tool responds to its version command:

```cmd
az --version
terraform --version
code --version
```

- Open the lab folder in VS Code so you can review and edit Terraform and PowerShell files side by side before running anything

---

## Step 2 — Understand the Lab Structure

Review the two main components before running anything.

- Open the lab folder in VS Code and identify the two file types:
  - **Terraform files** — define and provision the Azure infrastructure (VMs, networking, resource groups)
  - **PowerShell scripts** — configure the operating systems, promote the domain controller, create AD users and groups, configure file shares, and apply permissions
- Understand the execution order — Terraform runs first to build the environment, PowerShell runs second to configure it
- Do not run PowerShell scripts before Terraform has successfully provisioned the infrastructure

---

## Step 3 — Provision the Azure Infrastructure with Terraform

Use Terraform to create all required Azure resources automatically.

- Authenticate to Azure from the terminal:

```cmd
az login
```

- Navigate to the Terraform directory in VS Code's integrated terminal
- Initialize Terraform:

```cmd
terraform init
```

- Preview the resources that will be created:

```cmd
terraform plan
```

- Apply the configuration to provision the infrastructure:

```cmd
terraform apply
```

- Confirm when prompted by typing `yes`
- Wait for Terraform to complete provisioning
- Verify the following resources were created in Azure:
  - Domain controller VM
  - File server VM
  - Client workstation VM

> ⚠️ Do not proceed to Step 4 until `terraform apply` completes without errors.

---

## Step 4 — Run the PowerShell Configuration Script

Execute the configuration script to build the Active Directory environment on top of the provisioned infrastructure.

- Connect to the domain controller VM via RDP or Azure Bastion
- Open PowerShell as Administrator
- Run the configuration script:

```powershell
.\ConfigureLab.ps1
```

- Allow the script to complete all tasks in sequence:
  - Promote the VM to a domain controller
  - Create Active Directory users and security groups
  - Configure file shares on the file server
  - Apply share-level and NTFS permissions per group
- Confirm the script completes without errors before moving to validation

> ⚠️ Run Terraform and PowerShell in the correct order — infrastructure first, configuration second. Running the PowerShell script before the VMs exist will cause it to fail.

---

## Step 5 — Confirm the Access Model

Review the expected groups, users, and permissions before beginning validation.

| Group | Share | Permission Level |
|---|---|---|
| HR | HR | Read / Write |
| IT | IT | Full Control |
| Finance (Sarah) | Finance | Read / Write |
| Finance (Lisa) | Finance | Read-Only |
| Sales | Sales | Scoped access |

- Confirm that each user belongs to the correct AD group
- Confirm that each group is mapped to the correct file share
- Confirm that users should only be able to access shares assigned to their group

---

## Step 6 — Validate Sarah's Finance Access and HR Restriction

- RDP into the client workstation as **Sarah**
- Navigate to the **Finance** share
- Confirm Sarah can:
  - Create a new folder
  - Create a text document
  - Save changes successfully
- Attempt to open the **HR** share
- Confirm access is **denied** and the expected permission error message appears

> ✅ Expected result: Finance share accessible with read/write. HR share blocked with access denied message.

---

## Step 7 — Validate Lisa's Read-Only Finance Access

- RDP into the client workstation as **Lisa**
- Navigate to the **Finance** share
- Confirm Lisa can open and read existing files
- Attempt to delete a file
- Confirm deletion is **blocked** — Lisa has read-only permissions

> ✅ Expected result: Files readable. Deletion blocked by NTFS permissions.

---

## Step 8 — Validate HR Read/Write Access

- RDP into the client workstation using the **HR user** account
- Navigate to the **HR** share
- Confirm the HR user can:
  - Create a new folder
  - Create a text document
  - Save changes
  - Delete the document
- Confirm full read/write access within the HR share

> ✅ Expected result: All create, edit, and delete operations succeed within the HR share.

---

## Step 9 — Validate John Smith's Full Control Over the IT Share

- RDP into the client workstation as **John Smith**
- Navigate to the **IT** share
- Confirm John can:
  - Create folders
  - Create files
  - Edit file contents
  - Delete files
- Confirm the IT share grants **full control** as expected

> ✅ Expected result: All operations succeed with no restrictions.

---

## Step 10 — Validate Tom Davis's Denied Access to the Finance Share

- RDP into the client workstation as **Tom Davis**
- Attempt to open the **Finance** share
- Confirm access is **denied**
- Verify the system displays the expected permission error message

> ✅ Expected result: Finance share blocked. Permission error message displayed.

---

## Step 11 — Document Completion

- Record that all five user permission scenarios were validated successfully
- Note the full build and troubleshooting time (approximately 8 hours for initial build)
- Capture screenshots of each validation result as evidence
- Commit all Terraform and PowerShell files to the GitHub repository

---

## Cautionary Notes

- Ensure the Azure subscription has sufficient permissions and resource limits before running `terraform apply`
- Always run Terraform before PowerShell — the configuration script depends on the VMs existing
- Do not skip any validation step — each user account must be tested individually to confirm permissions
- If a user can access a share they should not, stop and review AD group membership and share permissions immediately
- Be careful when testing delete operations — do not remove files needed for subsequent validation steps
- Managed Azure VMs incur costs while running — destroy the environment after the lab is complete:

```cmd
terraform destroy
```

---

## Tips for Efficiency

- Keep all Terraform and PowerShell files organized in a single lab folder and open the entire folder in VS Code
- Use VS Code's integrated terminal to run Terraform commands without switching windows
- Test one user role at a time to isolate permission issues faster
- Capture screenshots during each validation step to make documentation easier
- Reuse the same validation pattern for every user: open share → perform allowed action → attempt restricted action → confirm result
- Save the `terraform.tfstate` file — it is required for `terraform destroy` to clean up resources correctly

---

## Validation Checklist

| User | Share Tested | Expected Result | Pass / Fail |
|---|---|---|---|
| Sarah | Finance | Read/write access ✓ | |
| Sarah | HR | Access denied ✓ | |
| Lisa | Finance (read) | Files readable ✓ | |
| Lisa | Finance (delete) | Deletion blocked ✓ | |
| HR user | HR | Read/write access ✓ | |
| John Smith | IT | Full control ✓ | |
| Tom Davis | Finance | Access denied ✓ | |

---

## Command Reference

| Tool | Command | Purpose |
|---|---|---|
| Azure CLI | `az login` | Authenticate to Azure |
| Terraform | `terraform init` | Initialize Terraform working directory |
| Terraform | `terraform plan` | Preview resources to be created |
| Terraform | `terraform apply` | Provision Azure infrastructure |
| Terraform | `terraform destroy` | Tear down all lab resources |
| PowerShell | `.\ConfigureLab.ps1` | Run the AD configuration script |

---

*Sean Keener · Azure Infrastructure Lab Series · [github.com/seankeener](https://github.com/seankeener)*
