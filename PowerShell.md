# PowerShell code snips

File selection function

    Function Get-FileName($initialDirectory)
    {
        [System.Reflection.Assembly]::LoadWithPartialName("System.windows.forms") | Out-Null

        $OpenFileDialog = New-Object System.Windows.Forms.OpenFileDialog
        $OpenFileDialog.initialDirectory = $initialDirectory
        # $OpenFileDialog.filter = "CSV (*.csv)| *.csv"
        $OpenFileDialog.ShowDialog() | Out-Null
        $OpenFileDialog.filename
    }

Load a CSV file

    $filename = Get-FileName

    function Load-CSVFile($filename){
      Write-Host "Loading File: $filename"
      $fileContent = Import-Csv -Path $filename
      Write-Host "Loading File Complete, $($fileContent.Count) entries loaded"
      return $fileContent
    }

Iterate over the CSV filename

    # Load the file
    $filename = Get-FileName()
    $list = Load-CSVFile()

    # Add a Results column
    $list = $list | Select *,"Result"

    foreach ($item in $list) {
        # reminder, $item is a reference so changes to it 
        # are reflectedin $list, just dont change the structure

        try {
            $item.Result = Some Command
        }
        catch{
            Write-Warning -Message "error"
            $item.Result = "error"
        }
    }

    # Save the results
    $list | Export-Csv -Path $filename -NoTypeInformation

Do Mass DNS Lookups

    # CSV File must have a column called Query with a list of queries
    $List = Load-CSVFile(Get-FileName())
    $List.Query | resolve-dnsname | where {$_.Section -eq "Answer"} | Select Name, IPAddress | Sort Name

List files and folders to a csv with atrribute data

    Get-ChildItem -Force -Recurse | Select-Object  Fullname, LastAccessTime, LastWriteTime, CreationTime, Mode, Length, @{N="Owner";E={ (Get-Acl).Owner }}, @{N="ACL";E={ (Get-Acl).Access | ConvertTo-Csv }} | Export-Csv ("C:\Collection\" + "DirectoryListing_"+(Get-Date -f yyyy-MM-dd_HH-mm-ss) + ".csv")

List folders only to a csv with atrribute data

    Get-ChildItem -Force -Recurse -Directory | Select-Object  Fullname, LastAccessTime, LastWriteTime, CreationTime, Mode, Length, @{N="Owner";E={ (Get-Acl).Owner }}, @{N="ACL";E={ (Get-Acl).Access | ConvertTo-Csv }} | Export-Csv ("C:\Collection\" + "DirectoryListing_D-only"+(Get-Date -f yyyy-MM-dd_HH-mm-ss) + ".csv")

Get a count of all members of an AD group recursive

    (Get-ADGroupMember -Identity 'Users' -Recursive).count

Open the Exchange Server On-Prem snapin from a powershell script and get version ect...

    Add-PSSnapin Microsoft.Exchange.Management.PowerShell.SnapIn
    Get-ExchangeServer | Format-List Name,Edition,AdminDisplayVersion,MitigationsApplied,MitigationsBlocked
    
Print all permissions on a o365 mailbox

    function Get-All-Permissions {
            param(
            [Parameter(Mandatory)]
            [string]$Mailbox
        )
        $AllFolderPermissions = @()
        $MBX= Get-Mailbox $Mailbox
        $MBXInboxRules = Get-InboxRule -Mailbox $Mailbox
        $MBXpermission = get-mailboxpermission $Mailbox | where {($_.User -notlike "NT AUTHORITY\SELF")}
        $MBXSendAs = Get-EXORecipientPermission $MBX | where {($_.Trustee -notlike "NT AUTHORITY\SELF")} | ft Identity, Trustee, AccessRights, AccessControlType, Inherited
        $MBXSendonBehalf = $MBX.GrantSendOnBehalfTo | where {($_)} | get-user | where {($_.RecipientType -notlike "MailUser")} | ft Name, DisplayName, RecipientType
        $MBXCalendarProcessing = Get-CalendarProcessing $Mailbox
        $MBXCalendarDelegates = $MBXCalendarProcessing.ResourceDelegates | where {($_)} | get-user | where {($_.RecipientType -notlike "MailUser")} | ft Name, DisplayName, RecipientType
        # get a list of all folders
        $MBXfolders=Get-EXOMailboxFolderStatistics $Mailbox | select FolderPath | where {($_.FolderPath -notlike "/Top of Information Store")}
        # Get Top permissions
        $allfolderpermissions += Get-EXOMailboxFolderPermission $Mailbox | where {($_.AccessRights -notlike "None")} | where {($_.User -notlike "NT AUTHORITY\SELF")}
        # Loop over every folder
        Foreach ($MBXfolder in $MBXfolders){
            try {
                $folder=$MBX.PrimarySmtpAddress + ":" + $MBXfolder.FolderPath -replace '/', '\'
                $folderpermissions= Get-EXOMailboxFolderPermission -Identity $folder -ErrorAction Stop | where {($_.AccessRights -notlike "None")} | where {($_.User -notlike "NT AUTHORITY\SELF")}
                $allfolderpermissions += $folderpermissions
            }
            catch {
                Write-Output "Error: failed to read permissions on folder: $folder"
                Continue
            }
        }
        Write-Output "========== Mailbox Settings ========="
        Write-Output "Mailbox: $($MBX.PrimarySmtpAddress)"
        Write-Output "UPN: $($MBX.UserPrincipalName)"   
        Write-Output "ForwardingSmtpAddress: $($MBX.ForwardingSmtpAddress)"
        Write-Output "DeliverToMailboxAndForward: $($MBX.DeliverToMailboxAndForward)"
        Write-Output ""
        Write-Output "========= Calendar Permissions ========="
        Write-Output "ForwardRequestsToDelegates: $($CalendarProcessing.ForwardRequestsToDelegates)"
        Write-Output "ResourceDelegates"
        Write-Output $MBXCalendarDelegates
        Write-Output ""
        Write-Output "========= Send As Permissions ========="
        Write-Output "These settings can be modified with the following command"
        Write-Output "#Remove-RecipientPermission -AccessRights SendAs -Identity $($MBX.PrimarySmtpAddress)"
        write-output $MBXSendAs
        write-output ""
        Write-Output "========= Send On Behalf Permissions ========="
        Write-Output "These settings can be modified with the following command"
        Write-Output "#Set-Mailbox $($MBX.PrimarySmtpAddress) -GrantSendOnBehalfTo @{Remove=""USER""}"
        write-output $MBXSendonBehalf
        write-output ""
        Write-Output "========= Mailbox Permissions ========="
        Write-Output "These settings can be modified with the following command"
        Write-Output "#Remove-MailboxPermission $($MBX.PrimarySmtpAddress) -AccessRights FullAccess -InheritanceType All"
        write-output $MBXpermission
        write-output ""
        Write-Output "========= Folder Permissions ========="
        Write-Output "These settings can be modified with the following command"
        Write-Output "#Remove-MailboxFolderPermission"
        Write-Output ($allfolderpermissions | ft)
        write-output ""
        Write-Output "========= Inbox Rules ========="
        Write-Output "These settings can be modified with the following command"
        Write-Output "#Remove-InboxRule"
        write-output ($MBXInboxRules | ft)
    }