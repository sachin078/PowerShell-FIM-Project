# PowerShell-FIM-Project
  [File Integrity Monitoring with PowerShell]

## Overview

This project demonstrates a simple File Integrity Monitoring (FIM) system using PowerShell. It provides the ability to collect a new baseline of file hashes and continuously monitor files for changes, notifying users of any alterations.

## PowerShell Functions

```powershell
Function Calculate-File-Hash($filepath) {
    $filehash = Get-FileHash -Path $filepath -Algorithm SHA512
    return $filehash
}
Function Erase-Baseline-If-Already-Exists() {
    $baselineExists = Test-Path -Path .\baseline.txt

    if ($baselineExists) {
        # Delete it
        Remove-Item -Path .\baseline.txt
    }
}


Write-Host ""
Write-Host "What would you like to do?"
Write-Host ""
Write-Host "    A) Collect new Baseline?"
Write-Host "    B) Begin monitoring files with saved Baseline?"
Write-Host ""
$response = Read-Host -Prompt "Please enter 'A' or 'B'"
Write-Host ""

if ($response -eq "A".ToUpper()) {
    # Delete baseline.txt if it already exists
    Erase-Baseline-If-Already-Exists

    # Calculate Hash from the target files and store in baseline.txt
    # Collect all files in the target folder
    $files = Get-ChildItem -Path .\FIM 

    # For each file, calculate the hash, and write to baseline.txt
    foreach ($f in $files) {
        $hash = Calculate-File-Hash $f.FullName
        "$($hash.Path)|$($hash.Hash)" | Out-File -FilePath .\baseline.txt -Append
    }
    
}

elseif ($response -eq "B".ToUpper()) {
    
    $fileHashDictionary = @{}

    # Load file|hash from baseline.txt and store them in a dictionary
    $filePathsAndHashes = Get-Content -Path .\baseline.txt
    
    foreach ($f in $filePathsAndHashes) {
         $fileHashDictionary.add($f.Split("|")[0],$f.Split("|")[1])
    }

    # Begin (continuously) monitoring files with saved Baseline
    while ($true) {
        Start-Sleep -Seconds 1
        
        $files = Get-ChildItem -Path .\FIM

        # For each file, calculate the hash, and write to baseline.txt
        foreach ($f in $files) {
            $hash = Calculate-File-Hash $f.FullName
            #"$($hash.Path)|$($hash.Hash)" | Out-File -FilePath .\baseline.txt -Append

            # Notify if a new file has been created
            if ($fileHashDictionary[$hash.Path] -eq $null) {
                # A new file has been created!
                Write-Host "$($hash.Path) has been created!" -ForegroundColor Green
            }
            else {

                # Notify if a new file has been changed
                if ($fileHashDictionary[$hash.Path] -eq $hash.Hash) {
                    # The file has not changed
                }
                else {
                    # File file has been compromised!, notify the user
                    Write-Host "$($hash.Path) has changed!!!" -ForegroundColor Yellow
                }
            }
        }

        foreach ($key in $fileHashDictionary.Keys) {
            $baselineFileStillExists = Test-Path -Path $key
            if (-Not $baselineFileStillExists) {
                # One of the baseline files must have been deleted, notify the user
                Write-Host "$($key) has been deleted!" -ForegroundColor DarkRed -BackgroundColor Gray
            }
        }
    }
}

```
<h3>DEMO</h3>

<b>The script starts by displaying a prompt to the user, asking whether they want to:

A) Collect a new baseline (hashes of files in a target folder) or

B) Begin monitoring files with a saved baseline. <b>

<img src="https://i.imgur.com/ZoqHxFL.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>

<br>
<b>The user's response is read, and based on the choice:

If 'A' is chosen, it erases the existing "baseline.txt" file (if it exists) and calculates the SHA512 hash for each file in the ".\FIM" directory. The file paths and corresponding hashes are stored in "baseline.txt."

If 'B' is chosen, it loads the baseline information from "baseline.txt" into a dictionary ($fileHashDictionary). It then enters a continuous monitoring loop where it periodically checks for changes in the files in the ".\FIM" directory. 

In our demo, we will select "B".
<b><br/>
<br>
<img src="https://i.imgur.com/ocOHNna.png" height="80%" width="80%" alt="Disk Sanitization Steps"/><br/>
<br>
<b>For each file in the directory, it calculates the current hash and compares it with the hash stored in the baseline.
If a file has been created, it notifies the user.
If a file has changed, it notifies the user that the file has been compromised.
It also checks if any files present in the baseline have been deleted and notifies the user if a deletion is detected. <b><br/>
<br>
<img src="https://i.imgur.com/1HvH67c.png" height="80%" width="80%" alt="Disk Sanitization Steps"/><br/>
<br>
<img src="https://i.imgur.com/4TstAkw.png" height="80%" width="80%" alt="Disk Sanitization Steps"/><br/>
<br>
<img src="https://i.imgur.com/4o66FDJ.png" height="80%" width="80%" alt="Disk Sanitization Steps"/><br/>
<br>
<img src="https://i.imgur.com/n42dsF6.png" height="80%" width="80%" alt="Disk Sanitization Steps"/><br/>
<br>
<b> Notification via Write-Host:

Notifications about file status changes (creation, modification, deletion) are displayed using Write-Host with different foreground and background colors to distinguish between different events.
Infinite Loop:

The script contains an infinite loop (while ($true)) that continuously monitors files. It uses Start-Sleep to introduce a one-second delay between iterations.
This script can be used as a basic file integrity monitoring tool, helping to detect changes in files within a specified directory over time. The baseline information serves as a reference for the initial state of files, and any subsequent changes trigger notifications. <b><br/>


