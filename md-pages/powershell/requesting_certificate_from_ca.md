```ps
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