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

# Installazione chocolatey(packet manager windows)
<!-- spazio -->
    Set-ExecutionPolicy Bypass -Scope Process -Force; [System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072; iex     ((New-Object System.Net.WebClient).DownloadString('https://community.chocolatey.org/install.ps1'))
<!-- spazio -->
# Installazione AD-Domain e setting del DNS

-  ### AD-Domain:
<!-- spazio -->
    Install-WindowsFeature AD-Domain-Services #IncludeManagementTools #comando per installare i servizi di AD-Domain
<!-- spazio -->
    Import-Module ADDSDeployment #Importato il modulo dell'AD con i vari comandi cmd 
<!-- spazio -->
    Install-ADDSForest #comando per installare e configurare una AD Forest
    
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
    Add-Computer -DomainName xyz.com -Credential xyz\Administrator -Force -Restart #mettere credenziali ed accedere
<!-- spazio -->
