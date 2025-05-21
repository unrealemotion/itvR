# Backup-FileSystemPermissions PowerShell Script

**Version:** 1.0 (as per script header, though iterative improvements have been made)
**Author:** Your Name/Company (as per script header)
**Date:** 2023-10-27 (as per script header)

## Overview

`Backup-FileSystemPermissions.ps1` is a robust PowerShell script designed to collect and back up Access Control Lists (ACLs) for specified file system paths. It meticulously captures comprehensive security information including Owner, Group, Discretionary Access Control List (DACL), and System Access Control List (SACL).

The script is built for resilience, logging errors for individual item failures without halting the entire backup process. It produces multiple output formats for comprehensive backup and easy review.

## Features

*   **Comprehensive ACL Backup:** Collects full security descriptors (Owner, Group, DACL, SACL).
*   **Flexible Input:** Accepts one or more file/folder paths.
*   **Recursive Mode:** Optionally collects ACLs for all child items.
*   **Timestamped Backups:** Outputs are saved in a timestamped subfolder (e.g., `Backup_YYYYMMDD_HHMMSS`) to prevent overwriting previous backups.
*   **Multiple Output Formats:**
    *   **CLIXML (`FullPermissionsBackup.clixml`):** Stores full `FileSystemSecurity` objects for each item. This is the primary file for restoration purposes.
    *   **CSV (`DACL_Report.csv`):** Provides a human-readable report of DACL entries (Permissions).
    *   **Text Log (`ProcessingLog.txt`):** Detailed log of the script's execution, including parameters, item-by-item status (success/failure with error messages), and a final summary.
*   **Advanced Error Handling:** Gracefully handles errors like "Access Denied" or "Path Not Found" for individual items, logs them, and continues processing.
*   **User Experience:**
    *   Supports `-Verbose` output for step-by-step console progress.
    *   Displays a progress bar during lengthy operations.
*   **Customizable Output Location:** Allows specifying a custom output directory.

## Prerequisites

*   **PowerShell:**
    *   Recommended: PowerShell 5.0 or newer (due to `[System.Collections.ArrayList]::new()` syntax. For older versions, these would need to be changed to `New-Object System.Collections.ArrayList`).
    *   The script *should* work on PowerShell 3.0+ if the `::new()` constructors are modified.
*   **Permissions:**
    *   **Run as Administrator:** Generally recommended for accessing system files/folders and for SACL retrieval.
    *   **"Manage auditing and security log" (SeSecurityPrivilege):** Required to read SACLs (audit rules). Without this, SACLs will not be backed up.
    *   **"Backup files and directories" (SeBackupPrivilege):** May be required to access certain files or folders that the user running the script wouldn't normally have direct read access to.

## How to Run

1.  Save the script as `Backup-FileSystemPermissions.ps1` (or any other `.ps1` name).
2.  Open a PowerShell console.
3.  Navigate to the directory where you saved the script.
4.  Execute the script with the required parameters.

    **Example:**
    ```powershell
    .\Backup-FileSystemPermissions.ps1 -Path "C:\Data\ProjectFolder" -Recursive -Verbose
    ```

## Parameters

*   `Path` (string[])
    *   **Mandatory:** Yes
    *   Specifies one or more file or folder paths for which to back up ACLs.
    *   Example: `-Path "C:\Folder1", "D:\Files\report.docx"`

*   `Recursive` (switch)
    *   **Mandatory:** No
    *   If present, the script will collect ACLs for all child items (files and folders) within the specified paths.
    *   Example: `-Recursive`

*   `OutputDirectory` (string)
    *   **Mandatory:** No
    *   Specifies the base directory where the timestamped backup subfolder will be created.
    *   Defaults to a subfolder named `PermissionBackups` in the script's current execution directory (e.g., `.\PermissionBackups`).
    *   Example: `-OutputDirectory "E:\Backups\ACL_Snapshots"`

*   Common Parameters:
    *   `-Verbose`: Enables detailed console output of the script's operations.
    *   `-Debug`, `-ErrorAction`, etc., are also supported as per standard PowerShell functions.

## Output Files

All output files are placed within a timestamped subfolder inside the specified (or default) `OutputDirectory`.
For example: `C:\Path\To\OutputDirectory\PermissionBackups\Backup_20231027_100000\`

1.  **`FullPermissionsBackup.clixml`**
    *   **Format:** XML (PowerShell CLIXML)
    *   **Content:** An array of PowerShell objects. Each object contains:
        *   `Path`: The full path to the file/folder.
        *   `AclObject`: The complete `System.Security.AccessControl.FileSystemSecurity` object (includes Owner, Group, DACL, SACL).
        *   `Sddl`: The Security Descriptor Definition Language (SDDL) string for the item.
        *   `RetrievedAt`: Timestamp of when the ACL was retrieved.
    *   **Purpose:** This is the primary backup file. It can be imported back into PowerShell (`Import-Clixml`) and used with `Set-Acl` to restore permissions.

2.  **`DACL_Report.csv`**
    *   **Format:** Comma Separated Values (CSV)
    *   **Content:** A table with detailed information for each Access Control Entry (ACE) in the DACL of successfully processed items. Columns include:
        *   `FolderPath`
        *   `IdentityReference` (User/Group)
        *   `FileSystemRights`
        *   `AccessControlType` (Allow/Deny)
        *   `IsInherited`
        *   `InheritanceFlags`
        *   `PropagationFlags`
    *   **Purpose:** Provides a human-readable overview of permissions (DACLs) for auditing or quick review.

3.  **`ProcessingLog.txt`**
    *   **Format:** Plain Text
    *   **Content:**
        *   Script start time, end time, and parameters used.
        *   A hierarchical (tree-like if `-Recursive`) list of all items processed:
            *   Full path of the item.
            *   Status: "SUCCESS" or "FAILURE".
            *   If "FAILURE", includes the specific error message.
        *   Summary: Total items scanned, number of successes, number of failures.
    *   **Purpose:** Detailed logging for troubleshooting and auditing the backup process.

## Example Usage

1.  **Backup ACLs for a single folder and its contents, with verbose output:**
    ```powershell
    .\Backup-FileSystemPermissions.ps1 -Path "E:\Shared\ImportantDocs" -Recursive -Verbose
    ```

2.  **Backup ACLs for multiple specific files and folders, output to a custom directory:**
    ```powershell
    .\Backup-FileSystemPermissions.ps1 -Path "C:\Data\File1.txt", "C:\Data\SensitiveFolder", "D:\Archive\OldReport.docx" -OutputDirectory "F:\SecurityBackups"
    ```

3.  **Backup ACLs for a top-level folder only (not recursive):**
    ```powershell
    .\Backup-FileSystemPermissions.ps1 -Path "C:\Program Files\MyApp"
    ```

## Restoring Permissions (Conceptual)

This script focuses on backup. To restore permissions, you would typically use the `FullPermissionsBackup.clixml` file.

**Conceptual Example (restore ACL for a single item):**
```powershell
# Import the backed-up ACLs
$BackedUpAcls = Import-Clixml -Path "C:\Path\To\Backup_YYYYMMDD_HHMMSS\FullPermissionsBackup.clixml"

# Find the ACL for a specific path you want to restore
$aclToRestore = $BackedUpAcls | Where-Object {$_.Path -eq "C:\Data\ProjectFolder\SpecificFile.txt"}

if ($aclToRestore) {
    try {
        Set-Acl -Path $aclToRestore.Path -AclObject $aclToRestore.AclObject -ErrorAction Stop
        Write-Host "Successfully restored ACL for $($aclToRestore.Path)"
    } catch {
        Write-Error "Failed to restore ACL for $($aclToRestore.Path). Error: $($_.Exception.Message)"
    }
} else {
    Write-Warning "ACL for 'C:\Data\ProjectFolder\SpecificFile.txt' not found in backup."
}
```
**Note:** Restoring permissions, especially recursively or in bulk, requires careful planning and thorough testing in a non-production environment.

## Troubleshooting

*   **Access Denied Errors:**
    *   Ensure you are running PowerShell as an Administrator.
    *   Verify the account running the script has the necessary permissions on the target files/folders.
    *   To retrieve SACLs, the account needs the "Manage auditing and security log" (SeSecurityPrivilege) right.
    *   The "Backup files and directories" (SeBackupPrivilege) right can also help bypass standard permission checks for backup purposes.
*   **Path Not Found Errors:**
    *   Double-check the paths provided to the `-Path` parameter for typos or incorrect locations.
*   **Output Directory Creation Failure:**
    *   Ensure the path specified in `-OutputDirectory` is valid and the script has write permissions to create subdirectories there.
*   **`Join-String` not found (Older PowerShell Versions):**
    *   This script has been updated to use the `-join` operator instead of `Join-String`, so this specific error should not occur with the provided version.
*   **`::new()` syntax errors (Older PowerShell Versions):**
    *   If running on PowerShell versions prior to 5.0, replace `[System.Collections.ArrayList]::new()` with `New-Object System.Collections.ArrayList` and `[System.Collections.Generic.List[hashtable]]::new()` with `New-Object "System.Collections.Generic.List[System.Collections.Hashtable]"`. The provided script uses `::new()`.

## Disclaimer

This script is provided "as-is" without any warranty. Always test backup and restore procedures in a non-production environment before relying on them for critical data. The author is not responsible for any data loss or system issues that may arise from its use.
