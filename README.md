# azure-germany
## Create a service principal with certificate authentication for deployment
### Login to your account
    Login-AzureRmAccount -EnvironmentName AzureGermanCloud

### Create a self signed certificate:
    $certificate = New-SelfSignedCertificate -CertStoreLocation "Cert:\LocalMachine\My" -Subject "CN=Deployment" -KeySpec KeyExchange

### Determine the certificate key:
    $certificateKey = [System.Convert]::ToBase64String($certificate.GetRawCertData())

### Create a new Active Directory application:
    $application = New-AzureRmADApplication -DisplayName "Deployment" -HomePage "https://www.yoursite.com" -IdentifierUris "https://www.yoursite.com/deployment" -KeyValue $certificateKey -KeyType AsymmetricX509Cert -EndDate $certificate.NotAfter -StartDate $certificate.NotBefore      

### Create a service principal for the application
    New-AzureRmADServicePrincipal -ApplicationId $application.ApplicationId

### Grant the service principal owner permisson on your subscription:
    New-AzureRmRoleAssignment -RoleDefinitionName Owner -ServicePrincipalName ($application.ApplicationId | select -ExpandProperty Guid)

### The deployment script can now use the service principal for authentication:
    Add-AzureRmAccount -ServicePrincipal -CertificateThumbprint <THUMBPRINT> -ApplicationId <APPID> -TenantId <TENANTID>

See also:
[Use Azure PowerShell to create a service principal to access resources](https://azure.microsoft.com/en-us/documentation/articles/resource-group-authenticate-service-principal/#get-tenant-id)