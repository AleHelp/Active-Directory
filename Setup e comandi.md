# Setup:

[VMware workstation 17](https://www.vmware.com/go/getworkstation-win)
<!-- spazio -->
[Virtual machine con al suo interno windows server core 2022](https://www.microsoft.com/it-IT/evalcenter/evaluate-windows-server-2022)
<!-- spazio -->
Config. tecniche:
  - 5.5 Gb ram
  - 4 processori
  - 70gb di storage
  - VMware tools installati
<!-- spazio -->
[Virtual machine con windows 10 enterprise](https://www.microsoft.com/it-it/evalcenter/download-windows-10-enterprise)
<!-- spazio -->
Config. tecniche:
  - 5 Gb ram
  - 4 processori
  - 70gb di storage
  - VMware tools installati

# Procedimenti per l'installazione
_Come una normale virtual machine montiamo le ISO, specifichiamo le caratteristiche hardware e le avviamo, al primo boot seguiamo passo passo l'installazione.
Nella windows server 2022 creiamo l'account Administrator con una semplice passwd="Passw0rd123" e lo stesso procedimento con l'altra macchina chiamando l'user localadmin
e con una passwd tipo="Admin123"_

# Comandi Powershell
_Qua sono riportati comandi utilizzati da me per debugging e conoscenza personale._
<!-- spazio -->
    Remove-ADObject -Identity "CN=Bob ob,CN=Users,DC=xyz,DC=com" -Recursive -Confirm:$false #Elimina un gruppo nell'AD
<!-- spazio -->
    Remove-ADObject -Identity "CN=Alice lice,CN=Users,DC=xyz,DC=com" -Recursive -Confirm:$false #rimuove un utente dell'AD
<!-- spazio -->
    Get-ADGroup | fl #lista tutti i gruppi
<!-- spazio -->
    Get-ADUsers | fl #lista tutti gli utenti
<!-- spazio -->
# Attivazione del PSRemoting
__Psremoting(Powershell remoting) è una funzionalità in powershell che permette di eseguire comandi Powershell da remoto, 
sfrutta il WinRM (Uguale all RDP ma più sicuro), per utilizzarlo va startato il servizio.__

- ### Comandi:
<!-- spazio -->
    Start-Service winRM #viene avviato il servizio WinRM
<!-- spazio -->
    Set-Item wsman:\localhost\Client\TrustedHosts -value <ip del server windows> #viene aggiunto l'indirizzo ip ai trustedhosts, stesso concetto delle 
    authorized_keys in ssh
<!-- spazio -->
    New-PSSession -ComputerName <ip del server windows> -Credential (Get-Credential) #stiamo avviando una sessione del PSRemoting e inseriamo le crendenziali del 
    windows server
<!-- spazio -->
    Enter-PSSession <numero sessione>
_Per velocizzare il processo possiamo creare una variabile d'ambiente e allocarci la PS-Session:_
<!-- spazio -->
     $id = New-PSSession <ip del server> -Credential (Get-Credential)
<!-- spazio -->
     Enter-PSSession $id
<!-- spazio -->
# Installazione chocolatey(packet manager windows)
<!-- spazio -->
    Set-ExecutionPolicy Bypass -Scope Process -Force; [System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072; iex     ((New-Object System.Net.WebClient).DownloadString('https://community.chocolatey.org/install.ps1'))
    #utilizzato come packet manager per installare git o vscode.
<!-- spazio -->
    Import-Module $env:ChocolateyInstall\helpers\chocolateyProfile.psm1 #modulo per le variabili d'ambiente
<!-- spazio -->
    refreshenv  #Aggiorna le variabili d'ambiente
<!-- spazio -->
# Installazione AD-Domain e setting del DNS

-  ### AD-Domain:
<!-- spazio -->
    Install-WindowsFeature AD-Domain-Services #IncludeManagementTools #comando per installare i servizi di AD-Domain
<!-- spazio -->
    Import-Module ADDSDeployment #Importato il modulo dell'AD con i vari comandi cmd 
<!-- spazio -->
    Install-ADDSForest #comando per installare e configurare una AD Forest
<!-- spazio -->    
-  ### DNS settings:
<!-- spazio -->
    Get-NetIPAddress #comando powershell per ottenere configurazione di rete
<!-- spazio -->
    Get-DnsClientCache #recupero della cache del DNS locale
<!-- spazio -->
    Get-DnsClientServerAddress #recupero delle configurzione di rete del DNS 
<!-- spazio -->
    Set-DnsClientServerAddress -InterfaceIndex <num interfaccia> - ServerAddress <indirizzo ip>
<!-- spazio -->
  _Dopo aver installato l'active directory l'ip del DNS sarà settato al 127.0.0.1, andrà cambiato con quello dell'interfaccia di rete principale._
<!-- spazio -->
# Entrare Nell'AD
<!-- spazio -->
_Prima cosa settare il DNS interno con l'indirizzo del Domain Controller._
<!-- spazio -->
     Set-DnsClientServerAddress -InterfaceIndex <num interfaccia> - ServerAddress <indirizzo ip DC>
<!-- spazio -->
_Per entrare nell'AD cercare la voce __accedi all'azienda o all'istituto di istruzione__, cliccare connetti e poi selezionare la voce __aggiungi a dominio di Active Directory locale__. o eseguire il comando in powershell:_ 
<!-- spazio -->
    Add-Computer -DomainName <nome_dominio> -Credential <nome account a dominio> -Force -Restart #immettere credenziali ed accedere
<!-- spazio -->
# Configurazione user e gruppi:
<!-- spazio -->
_La configurazione degli utenti avviene tramite un json file_
<!-- spazio -->
    {
        "domain": "xyz.com",
        "groups": [
            {
                "name": "Employees"
            }
        ],
        "users": [
            {
                "name": "Alice lice",
                "password": "Anyth1ng1234",
                "groups": [
                    "Employees"
                ]
            },
            {
                "name": "Bob ob",
                "password": "P@ssw0rdABC",
                "groups": [
                    "Employees"
                ]
            }
        ]
    }
<!-- spazio -->
_Si continua poi con uno script in powershell dove verrà mandato in input il JSON file e da li verranno generati credenziali, nome,cognome,username,gruppo ecc..._ 
<!-- spazio -->
    Set-ExecutionPolicy RemoteSigned #bisogna cambiare le policy per runnare gli script
<!-- spazio -->
_Script:_
<!-- spazio -->
    param([Parameter(Mandatory=$true)] $JSONfile)
    
    function CreateADGroup(){
        param([Parameter(Mandatory=$true)] $groupObject)
    
        $name = $groupObject.name
        New-ADGroup -name $name -GroupScope Global
    }
    
    function CreateADUser() {
        param([Parameter(Mandatory=$true)] $userObject)
    
        #estrae i nomi dal json
        $name = $userObject.name
        $password = $userObject.password
    
        #generazione del cognome e username
        $firstname , $lastname = $name.split(" ")
        $username = ($firstname[0] + $lastname).ToLower()
        $SamAccountName = $username
        $principalname = $username
    
        #crea l'oggetto AD user
        New-ADUser -Name "$name" -GivenName $firstname -Surname $lastname -SamAccountName $SamAccountName -UserPrincipalName $principalname@$Global:Domain -      
        AccountPassword (ConvertTo-SecureString $password -AsPlainText -Force) -PassThru | Enable-ADAccount
    
        #aggiunge l'user al gruppo
        foreach($group_name in $userObject.groups){
            try{
                Get-ADGroup -Identity "$group_name"
                Add-ADGroupMember -Identity $group_name -Members $username
            }
            catch [Microsoft.ActiveDirectory.Management.ADIdentityNotFoundException]
            {
                Write-Warning "User $name NOT added to group $group_name because it doesn't exists"
            }
        }
    }
    
    $json = (Get-Content $JSONfile | ConvertFrom-JSON)
    $Global:Domain = $json.domain
    
    foreach($group in $json.groups){
        CreateADGroup $group
    }
    
    
    foreach($user in $json.users){
        CreateADUser $user
    }
    
