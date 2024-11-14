#As an S6 I often got sent reports of computers that were out of date. Sometimes the comptuers were even on the network but 
#missing updates for some reason. This script would help get info about how out of date the computer is, and the contact info of
#the last user who logged into it in hopes that they can be a POC to diagnose and solve the problem.

#Receive Computer Names by prompt
param(
[string]$CNameInput=(Read-Host "Enter Computer Names, separated by commas")
)
#Alternate method: Receive computer names by CSV
#Write that later

#Alternate Method 2: Make this a function. I should probably get it out there first.

#Create Array of Computer Names
$CNameArray = $CNameInput -split "\,"


#Create File Path
$filepath = ".\ActivityReports\activityreport_$(Get-Date -format 'yyy-mm-dd').txt"

foreach($C in $CNameArray) {
    $session = New-PSSession -ComputerName $C
    $osVersion = Invoke-Command -Session $session -ScriptBlock { Get-CimInstance -ClassName Win32_OperatingSystem | Select-Object -ExpandProperty Version }
    $lastUserName = Invoke-Command -Session $session -ScriptBlock { Get-ChildItem "c:\Users" | Sort-Object LastWriteTime -Descending | Select-Object -ExpandProperty Name -first 1}
    $lastUser = Get-ADUser -Identity $lastUserName
    $LastUserContact = get-ADUser -IDENTITY $lastUser -Properties mobile | Select-Object -ExpandProperty mobile
    $output = [PSCustomObject]@{
    ComputerName = $C
    WinVersion = $osVersion
    Name = $lastUserName
    ContactInfo = $LastUserContact
    }
    $output | Out-File -FilePath $filepath -Append
    Get-PSSession | Remove-PSSession
}


#Things to do next: Add last GP-Update


<#
$allUsers = Get-ADUser -Filter *
foreach($u in $allUsers) {
$ranNum = Get-Random -Minimum 1000000000 -Maximum 9999999999
Set-ADUser $u -MobilePhone $ranNum
}

I used this to add cell numbers to ad users for this script for troubleshooting in my lab environment

#>