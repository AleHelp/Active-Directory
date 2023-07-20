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

[Virtual machine con kali linux](https://www.kali.org/get-kali/#kali-installer-images)
<!-- spazio -->
Config. tecniche:
  - 5 Gb ram
  - 4 processori
  - 40gb di storage
  - VMware tools installati

___FARE DEGLI SNAPSHOT DELLE VM OGNI TANTO___

# Procedimenti per l'installazione
_Come una normale virtual machine montiamo le ISO, specifichiamo le caratteristiche hardware e le avviamo, al primo boot seguiamo passo passo l'installazione.
Nella windows server 2022 creiamo l'account Administrator con una semplice passwd="Passw0rd123" e lo stesso procedimento con l'altra macchina chiamando l'user localadmin
e con una passwd tipo="Admin123"_
<!-- spazio -->
__N.B Nella VM avente Windows 10 enterprise selezionamo di montare dopo il sistema operativo, infatti dopo aver installato tutto andando in setting->cd/dvd(sata) specifichiamo la ISO da montare,riavviamo la VM e montiamo il tutto.__
 
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
    net user /domain o net group /domain #comandi terminale per ricavare informazioni su gruppi o utenti del domain.
<!-- spazio -->
    Get-DnsClientCache #recupero della cache del DNS locale
<!-- spazio -->
# Attivazione del PSRemoting
__Psremoting(Powershell remoting) è una funzionalità in powershell che permette di eseguire comandi Powershell da remoto, 
sfrutta il WinRM (Uguale all RDP ma più sicuro), per utilizzarlo va startato il servizio.__

- ### Comandi:
_(sulla windows 10 enterprise)_
<!-- spazio -->
    Start-Service winRM #viene avviato il servizio WinRM
<!-- spazio -->
    Set-Item wsman:\localhost\Client\TrustedHosts -value <ip del server windows> #viene aggiunto l'indirizzo ip ai trustedhosts, stesso concetto delle 
    authorized_keys in ssh
<!-- spazio -->
    $creds = (Get-Creds) # salviamo le credenziali del domain controller in una variabile 
<!-- spazio -->
     $id = New-PSSession <ip del server> -Credential $creds #salviamo tutto il comando in una variabile
<!-- spazio -->
     Enter-PSSession $id #comando per entrare nella sessione
<!-- spazio -->
      Copy-Item <Path del file da copiare> -ToSession <variabile d'ambiente che contienere il comando New-PSSession> <Destinaziona all'interno del DC dove copiare il file>
<!-- spazio -->
# Installazione chocolatey(packet manager windows)
<!-- spazio -->
    Set-ExecutionPolicy Bypass -Scope Process -Force; [System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072; iex     ((New-Object System.Net.WebClient).DownloadString('https://community.chocolatey.org/install.ps1'))
    #utilizzato come packet manager per installare git o vscode.
<!-- spazio -->
    Import-Module $env:ChocolateyInstall\helpers\chocolateyProfile.psm1 #modulo per le variabili d'ambiente
<!-- spazio -->
    choco install vscode #comando per installare un edito in questo caso visual studio code
<!-- spazio -->
    refreshenv  #Da utilizzare dopo aver installato un programma con choco
<!-- spazio -->
# Installazione AD-Domain e setting del DNS
_(sulla windows server core 2022 aka Domain Controller)_
<!-- spazio -->
-  ### AD-Domain:
<!-- spazio -->
    Install-WindowsFeature AD-Domain-Services #IncludeManagementTools #comando per installare i servizi di AD-Domain
<!-- spazio -->
    Import-Module ADDSDeployment #Importato il modulo dell'AD con i vari comandi cmd 
<!-- spazio -->
    Install-ADDSForest #comando per installare e configurare una AD Forest
<!-- spazio -->    
-  ### DNS settings:
 _Dopo aver installato l'active directory l'ip del DNS sarà settato al 127.0.0.1, andrà cambiato con quello dell'interfaccia di rete principale, cioè con l'ip del domain controller stesso_
<!-- spazio -->
    Get-NetIPAddress #comando powershell per ottenere configurazione di rete
<!-- spazio -->
    Get-DnsClientServerAddress #recupero delle configurzione di rete del DNS 
<!-- spazio -->
    Set-DnsClientServerAddress -InterfaceIndex <num interfaccia> - ServerAddress <indirizzo ip>
<!-- spazio -->
# Entrare Nell'AD
<!-- spazio -->
_Nella win 10 Enterprise prima cosa da settare è il DNS interno con l'indirizzo del Domain Controller._
<!-- spazio -->
     Set-DnsClientServerAddress -InterfaceIndex <num interfaccia> - ServerAddress <indirizzo ip DC>
<!-- spazio -->
_Per entrare nell'AD cercare la voce __accedi all'azienda o all'istituto di istruzione__, cliccare connetti e poi selezionare la voce __aggiungi a dominio di Active Directory locale__. o eseguire il comando in powershell:_ 
<!-- spazio -->
    Add-Computer -DomainName <nome_dominio> -Credential <nome account all'interno del domain controllwe> -Force -Restart #immettere credenziali ed accedere
<!-- spazio -->
# Configurazione user e gruppi:
<!-- spazio -->
__Durante la serie la creazione degli utenti, gruppi evolve in steps, quindi verrannò messi tutti i codici step by step.
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
_Si continua poi con uno script in powershell dove verrà mandato in input il JSON file e da li verranno generati credenziali, nome,cognome,username,gruppo ecc... in maniera quasi automatica_ 
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
 <!-- spazio -->
_Seconda configurazione più articolata, che permette un'ulteriore automazione, cioè generare il JSON degli utenti senza farlo a mano._
<!-- spazio -->
__N.B: Creare 4 .txt file contenenti Nomi,Cognomi,Gruppi e Password saranno utilizzati dal seguente script per creare il json.__
<!-- spazio -->
    #prende un json file in input dove verrannò messi gli user creati
    param([Parameter(Mandatory=$true)] $OutputJSONFile)
    
    #preso il contenuto dai vari file e converitti in array
    $group_names = [System.Collections.ArrayList](Get-Content  "C:\Users\localadmin\Desktop\Data\group_names.txt") #DA CAMBIARE
    $first_names = [System.Collections.ArrayList](Get-Content  "C:\Users\localadmin\Desktop\Data\first_names.txt")  #DA CAMBIARE
    $last_names = [System.Collections.ArrayList](Get-Content   "C:\Users\localadmin\Desktop\Data\last_names.txt")  #DA CAMBIARE
    $passwords = [System.Collections.ArrayList](Get-Content  "C:\Users\localadmin\Desktop\Data\passwords.txt")  #DA CAMBIARE
    
    #creazione lista gruppi e nomi
    $groups = @()
    $users = @()
    $num_groups = 10
    #creazione hashtable contenenti i gruppi
    for ($i = 0; $i -lt $num_groups; $i++){
        $new_group = (Get-Random  -InputObject $group_names)
        $group = @{"name" = "$new_group"}
        $groups += $group
        $group_names.Remove($new_group)
    
    }
    
    $users = @()
    $num_users = 10

    #creazione hashtable dei nomi
    for ($i = 0; $i -lt $num_users; $i++){
        $first_name = (Get-Random  -InputObject $first_names)
        $last_name = (Get-Random  -InputObject $last_names)
        $password = (Get-Random  -InputObject $passwords)
    
        $new_user = @{
            "name"="$first_name $last_name"
            "password"="$password" 
            "group" = @( (Get-Random -InputObject $groups).name )
        }
        $users += $new_user
        $first_names.Remove($first_name)
        $last_names.Remove($last_name)
        $passwords.Remove($password)
    }

    #hashtable finale per dominio gruppi e utenti
    @{
        "domain" = "xyz.com"
        "groups" = $groups
        "users" = $users
    } | ConvertTo-Json | Out-File $OutputJSONFile
<!-- spazio -->
_Ultima configurazione, con il JSON creato sopra, lo passiamo allo script sottostante che si occuperà di creare i: gruppi,user e relative password._
<!-- spazio -->
ES per avviare lo script:
<!-- spazio -->
    .\gen_ad.ps1   \out.json  $undo #se viene inserito il  parametro $Undo invece di creare gli utenti verranno eliminati
<!-- spazio -->
    param(
        [Parameter(Mandatory=$true)] $JSONfile,#viene passato in input il JSON file avente i vari utentiu, gruppi e password
        [switch]$Undo#undo utilizzato per elimiare user e gruppi
    )
    
    #crea un ADGroup con scopo Global
    function CreateADGroup(){
        param([Parameter(Mandatory=$true)] $groupObject)
    
        $name = $groupObject.name
        New-ADGroup -name $name -GroupScope Global
    }
    
    #funzione per rimuove ADGroup
    function RemoveADGroup(){
        param( [Parameter(Mandatory=$true)] $groupObject )
    
        $name = $groupObject.name
        Remove-ADGroup -Identity $name -Confirm:$False
    }
    
    #Crea un ADUser con scopi globali
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
        New-ADUser -Name "$name" -GivenName $firstname -Surname $lastname -SamAccountName $SamAccountName -UserPrincipalName $principalname@$Global:Domain -AccountPassword (ConvertTo-SecureString $password -AsPlainText -Force) -PassThru | Enable-ADAccount #questo comando deve essere tutto su una linea per il corretto funzionamento
    
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
    
    #funzione per eliminare gli utenti
    function RemoveADUser(){
        param( [Parameter(Mandatory=$true)] $userObject )
    
        $name = $userObject.name
        $firstname, $lastname = $name.Split(" ")
        $username = ($firstname[0] + $lastname).ToLower()
        $samAccountName = $username
        Remove-ADUser -Identity $samAccountName -Confirm:$False
    }
    
    #funzione per eliminare le restrizioni della password
    function WeakenPasswordPolicy(){
        secedit /export /cfg C:\Windows\Tasks\secpol.cfg
        (Get-Content C:\Windows\Tasks\secpol.cfg).replace("PasswordComplexity = 1", "PasswordComplexity = 0").replace("MinimumPasswordLength = 7", "MinimumPasswordLength = 1") | Out-File C:\Windows\Tasks\secpol.cfg
        secedit /configure /db c:\windows\security\local.sdb /cfg C:\Windows\Tasks\secpol.cfg /areas SECURITYPOLICY
        rm -force C:\Windows\Tasks\secpol.cfg -confirm:$False
    }
    
    #funzione che rafforza i criteri di sicurezza della password
    function StrenghtenPasswordPolicy(){
        secedit /export /cfg C:\Windows\Tasks\secpol.cfg
        (Get-Content C:\Windows\Tasks\secpol.cfg).replace("PasswordComplexity = 1", "PasswordComplexity = 0").replace("MinimumPasswordLength = 1", "MinimumPasswordLength = 7") | Out-File C:\Windows\Tasks\secpol.cfg
        secedit /configure /db c:\windows\security\local.sdb /cfg C:\Windows\Tasks\secpol.cfg /areas SECURITYPOLICY
        rm -force C:\Windows\Tasks\secpol.cfg -confirm:$False
    }
    
    $json = (Get-Content $JSONfile | ConvertFrom-JSON) 
    $Global:Domain = $json.domain
    
    #se viene zelezionato undo, gli user e gruppi nel json saranno eliminati senno' no
    if ( -not $undo){
                
        WeakenPasswordPolicy
    
        foreach($group in $json.groups){
            CreateADGroup $group
        }
    
        foreach($user in $json.users){
                CreateADUser $user
            }
    }else{
    
        StrenghtenPasswordPolicy
    
        foreach($user in $json.users){
                RemoveADUser $user
            }
    
        foreach($group in $json.groups){
                RemoveADGroup $group
            }
        }
<!-- spazio -->
_Una volta salvati gli script tramite PSRemoting andrà passato al Windows server core 2022(Domain Controller) il JSON creato con lo script e lo script che crea utenti,gruppi e password con il JSON generato._
<!-- spazio -->
# Utilizzo di CrackMapexec
_tool utilizzato nel mondo del pentesting per il crack di credenziali su diversi protocolli(SSH,FTP,LDAP,SMB)_
_Passi generali:_ 
<!-- spazio -->
- Creazione di una wordlist di utenti
<!-- spazio -->
- Creazione delle prime 1000 password di RockYou.txt
<!-- spazio -->
      gunzip /usr/share/wordlists/rockyou.txt.gz
<!-- spazio -->
      head -n 1000 /usr/share/wordlists/rockyou.txt > passwords.txt
- utilizzo di crackmapexec con diversi comandi:
<!-- spazio -->
      crackmapexec <protocollo> <target> -u <wordlist utenti> -p <wordlist password>
<!-- spazio -->
      crackmapexec <protocollo> <target> -u <wordlist utenti> -p <wordlist password> -groups #enumerazione dei gruppi
<!-- spazio -->
      crackmapexec <protocollo> <target> -u <wordlist utenti> -p <wordlist password> -users #enumerazione utenti
  <!-- spazio -->
  __I comandi sono molteplici è bene visionare l'help per saperne di più.__
<!-- spazio -->







