# Queryrun_to_SFTP

#runs sql query and returns a csv file of the results
Function Query-SQL {
    [CmdletBinding()]
    param(

        [Parameter(Position=0, Mandatory)]
        [ValidateNotNullorEmpty()]
        [string[]]$Variable_for_query
    )

    BEGIN {
    $Query = @"
    <# insert query here #>
"@
    }PROCESS {
	<#input server nbame and database name#>
        $Result = Invoke-Sqlcmd -Query $query -ServerInstance Servername -Database "database name here" -QueryTimeout 900 -ErrorAction Stop
    }
    END {

    <# Outputs unformated data into csv file#>
     $Result| Out-File C:\Users\Test.csv

    }
}

#transfers file to sftp folder and backup location
Function SFTPmove()
{

    # Set the credentials
    $Password = ConvertTo-SecureString 'password here' -AsPlainText -Force
    $Credential = New-Object System.Management.Automation.PSCredential ('account name here', $Password)

    # Set local file path and SFTP path
    $FilePath = "C:\Users\Test.csv"
    $SftpPath = '/in/archive'
    $BackupPath = "Z:\User\Backup"

    # Set the IP of the SFTP server
    $SftpIp = 'Ip address for server'


    # Establish the SFTP connection
    $ThisSession = New-SFTPSession -ComputerName $SftpIp -Credential $Credential

    # Upload the file to the SFTP path
    Set-SFTPFile -SessionId ($ThisSession).SessionId -LocalFile $FilePath -RemotePath $SftpPath

    #Disconnect all SFTP Sessions
    Get-SFTPSession | % { Remove-SFTPSession -SessionId ($_.SessionId) }

    # Copy the file to the SMB location
    Copy-Item -Path $FilePath -Destination $BackupPath
}

#main
Query-SQL -Variable 'variable content here'
SFTPmove
