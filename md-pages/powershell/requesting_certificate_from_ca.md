# Lookups
```powershell
# Find all available Certificate Authorities and their allowed templates
function Write-SupportedTemplatesPerCA {
    $rootDSE = [System.DirectoryServices.DirectoryEntry]'LDAP://RootDSE'
    $searchBase = [System.DirectoryServices.DirectoryEntry]"LDAP://$($rootDSE.configurationNamingContext)"
    $CAs = [System.DirectoryServices.DirectorySearcher]::new($searchBase,'objectClass=pKIEnrollmentService').FindAll()

    foreach ($CA in $CAs){
        Write-Host $CA.Properties.name -ForegroundColor Green
        foreach ($Template in $CA.Properties.certificatetemplates){
            Write-Host $Template
        }
    }
}
```

# Test CIM Connectivity with and without SSL
```powershell
$computername = "vmnaam"
$username = "testadmin"
$password = "****" | ConvertTo-SecureString -AsPlainText -Force # unsafeeee
 
# Make credential (unsafe!)
$cred = New-Object -typename System.Management.Automation.PSCredential -argumentlist $username, $password
 
# CIM
# After successful connection, try removing -SkipCACheck and/or -UseSsl
# And see when the connection fails
$sessionOptions = New-CimSessionOption -SkipCACheck -UseSsl
$cs = New-CimSession -Credential $cred -ComputerName $computername -SessionOption $sessionOptions
 
# Use cmdlet that accepts cimsession argument
Get-CimInstance Win32_DiskDrive -CimSession $cs
```