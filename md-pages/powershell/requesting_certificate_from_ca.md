# WinRM HTTPS Listener
This article will explain how to setup WinRM HTTPS using only powershell.

To enable the HTTPS listener in WinRM, you'll need to bind a certificate. In a stand-alone environment you can just use a self-signed certificate, like this:

``` powershell
# Read out Hostname and FQDN
$hostname = ($env:COMPUTERNAME).ToLower()

# Make a self-signed certificate
$cert = New-SelfSignedCertificate -Subject "CN=$hostname" -DnsName $hostname -CertStoreLocation Cert:\LocalMachine\My 

# Setup WinRM HTTPS Listener
winrm create winrm/config/Listener?Address=*+Transport=HTTPS "@{Hostname=`"$hostname`"; CertificateThumbprint=`"$($cert.Thumbprint)`"}"

# Not neccessary in most cases, but just to be sure:
netsh advfirewall firewall add rule name="Windows Remote Management (HTTPS-In)" dir=in action=allow protocol=TCP localport=5986
```

Note that when you use this solution, you will need to disable the certificate check whenever you connect via https, because there is no chain of authority linking back to the connecting client.

In a domain-joined environment, you might thus instead want to use a certificate that is signed by your PKI server (CA). 
In this article we'll just assume this is all configured, and the CA can be found from the host on which we want to setup WinRM.

## Lookup available CA and its templates
In certmgr.msc, you'll only find the certificate templates that you have access to.
The names listed are also not identical to the names that you have to use. (Mostly missing spaces causing this).

To find all available Certificate Authorities and their allowed templates (with actual names), do the following:

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

Write-SupportedTemplatesPerCA
```

## Request the CA certificate & Setup WinRM HTTPS Listener

``` powershell
# Read out Hostname and FQDN
$hostname = ($env:COMPUTERNAME).ToLower()
$domain_suffix = '<contoso.com>'
$fqdn = "$hostname.$domain_suffix"

# Request Certificate from template
$cert = Get-Certificate -Template "<YourTemplateName>" -SubjectName "CN=$fqdn" -DnsName $fqdn, $hostname -CertStoreLocation Cert:\LocalMachine\My

# Setup WinRM HTTPS Listener
winrm create winrm/config/Listener?Address=*+Transport=HTTPS "@{Hostname=`"$fqdn`"; CertificateThumbprint=`"$($cert.Thumbprint)`"}"

# Not neccessary in most cases, but just to be sure:
netsh advfirewall firewall add rule name="Windows Remote Management (HTTPS-In)" dir=in action=allow protocol=TCP localport=5986
```

## Test CIM Connectivity with and without SSL
One important way to test whether your authentication is working over SSL is by testing the CIM connection.

Below code should work, even over the HTTP listener. Note also the `-SkipCACheck` parameter, this relates to issue with self-signed certificates that we talked about earier.
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

Then, (after having run above code), run the following and check whether you get the same output (this should work for self-signed and CA certificates):
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

