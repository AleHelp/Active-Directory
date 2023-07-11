# Setup:

VMware workstation 17
<!--spazio-->
[Virtual machine con al suo interno windows server core 2022](https://www.microsoft.com/it-IT/evalcenter/evaluate-windows-server-2022)
Config. tecniche:
  - 5.5 Gb ram
  - 4 processori
  - 70gb di storage
  - VMware tools installati
<!--spazio-->
[Virtual machine con windows 10 enterprise](https://www.microsoft.com/it-it/evalcenter/download-windows-10-enterprise)
Config. tecniche:
  - 5 Gb ram
  - 4 processori
  - 70gb di storage
  - VMware tools installati

#Procedimenti per l'installazione
_Come una normale virtual machine montiamo le ISO, specifichiamo le caratteristiche hardware e le avviamo, al primo boot seguiamo passo passo l'installazione.
Nella windows server 2022 creiamo l'account Administrator con una semplice passwd="Passw0rd123" e lo stesso procedimento con l'altra macchina chiamando l'user localadmin
e con una passwd tipo="Admin123"_

#attivazione del PSRemoting
__Psremoting(Powershell remoting) è una funzionalità in powershell che permette di eseguire comandi Powershell da remoto, 
sfrutta il WinRM (Uguale all RDP ma più sicuro), per utilizzarlo va startato il servizio.__

##Comandi:

    Start-Service winRM #viene avviato il servizio WinRM
    <!--spazio-->

    Set-Item wsman:\localhost\Client\TrustedHosts -value <ip del server windows> #viene aggiunto l'indirizzo ip ai trustedhosts, stesso concetto delle authorized_keys in ssh
    <!--spazio-->

    New-PSSession -ComputerName <ip del server windows> -Credential (Get-Credential) #stiamo avviando una sessione del PSRemoting e inseriamo le crendenziali del windows server
    <!--spazio-->

    Enter-PSSession <numero sessione>



