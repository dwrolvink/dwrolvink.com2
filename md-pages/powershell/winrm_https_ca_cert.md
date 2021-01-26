# Lookups
Find all available Certificate Authorities and their allowed templates

```powershell
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

# Setup WinRM HTTPS Listener

``` powershell
# Read out Hostname and FQDN
$hostname = ($env:COMPUTERNAME).ToLower()
$domain_suffix = 'contoso.com'
$fqdn = "$hostname.$domain_suffix"

# Request Certificate from template
$cert = Get-Certificate -Template "YourTemplateName" -SubjectName "CN=$fqdn" -DnsName $fqdn, $hostname -CertStoreLocation Cert:\LocalMachine\My

# If you don't use a CA, just make a self-signed certificate
#$cert = New-SelfSignedCertificate -Subject "CN=$fqdn" -DnsName $fqdn, $hostname -CertStoreLocation Cert:\LocalMachine\My 

# Setup WinRM HTTPS Listener
winrm create winrm/config/Listener?Address=*+Transport=HTTPS "@{Hostname=`"$fqdn`"; CertificateThumbprint=`"$($cert.Thumbprint)`"}"

# Not neccessary in most cases, but just to be sure:
netsh advfirewall firewall add rule name="Windows Remote Management (HTTPS-In)" dir=in action=allow protocol=TCP localport=5986
```

# Test CIM Connectivity with and without SSL
One important way to test whether your authentication is working over SSL is by testing the CIM connection.

Below code should work, even over the HTTP listener. 
```powershell
$computername = "<computername>"
$username = "testadmin"
$password = "****" | ConvertTo-SecureString -AsPlainText -Force # unsafeeee
 
# Make credential (unsafe!)
$cred = New-Object -typename System.Management.Automation.PSCredential -argumentlist $username, $password
 
# CIM - HTTP
$sessionOptions = New-CimSessionOption -SkipCACheck
$cs = New-CimSession -Credential $cred -ComputerName $computername -SessionOption $sessionOptions
 
# Use cmdlet that accepts cimsession argument
Get-CimInstance Win32_DiskDrive -CimSession $cs
```

Then, (after having run above code), run the following and check whether you get the same output:
```powershell
# CIM - HTTPS - Skip CA check
$sessionOptions = New-CimSessionOption -SkipCACheck -UseSsl
$cs = New-CimSession -Credential $cred -ComputerName $computername -SessionOption $sessionOptions
 
# Use cmdlet that accepts cimsession argument
Get-CimInstance Win32_DiskDrive -CimSession $cs
```

And finally:
```powershell
# CIM - HTTPS
$sessionOptions = New-CimSessionOption -UseSsl
$cs = New-CimSession -Credential $cred -ComputerName $computername -SessionOption $sessionOptions
 
# Use cmdlet that accepts cimsession argument
Get-CimInstance Win32_DiskDrive -CimSession $cs
```

