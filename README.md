# ** Script D'exportation de plusieurs VM sur ESXi / Vcenter **

## Activation de l'éxecution de script powershell

```powershell
Set-ExecutionPolicy Unrestricted -Scope CurrentUser -Force
```

## Installation des modules VMWare PowerCLI

```powershell
Install-Module -Name VMware.PowerCLI -Scope AllUsers
```

### Installation du module en mode offline

#### Téléchargement du module (depuis internet)
```powershell
Save-Module -Name VMware.PowerCLI -Path C:\Users\Administrateur\Desktop\PowerCLI
```
Vous allez vous retrouver avec les dossiers contenant les différents modules.

![](./fichiers.jpg)

**Copiez les dossiers des modules dans l'un des trois dossiers suivants, en fonction du type d'installation que vous souhaitez réaliser :**

- le dossier "C:\Users\[nom de l'utilisateur actuel]\Documents\WindowsPowerShell\Modules" pour les modules installés pour l'utilisateur actuel.
- le dossier "C:\Program Files\WindowsPowerShell\Modules" pour les modules installés sur l'ordinateur pour tous les utilisateurs.
- le dossier "C:\Windows\System32\WindowsPowerShell\v1.0\Modules" pour les modules installés par défaut pour tous les utilisateurs de l'ordinateur.

## Vérification des modules précédemment installé

```powershell
Get-Module -Name VMware.PowerCLI -ListAvailable
```

## Script Powershell D'exportation des machines virtuelle :

```powershell
Set-PowerCLIConfiguration -InvalidCertificateAction Ignore -Confirm:$false
Connect-VIServer -Server 127.0.0.1 -User administrator@vsphere.local -Password toto

Get-VM | Select -ExpandProperty Name | Out-String | Format-list | Out-File C:\Temp\dump.txt

$vm_names = Get-Content C:\Temp\dump.txt

foreach ($vm_name in $vm_names)
{
  Get-VM -Name $vm_name | Shutdown-VMGuest -confirm:$false
  Get-VM -Name $vm_name | Get-CDDrive | Set-CDDrive -NoMedia -confirm:$false
  Get-VM -Name $vm_name | Export-VApp -Destination 'C:\Temp' -Format OVF -CreateSeparateFolder:$false
}
```

Credit : https://angrysysadmins.tech/index.php/2020/10/grassyloki/powercli-basic-scripts-bulk-export-all-virtual-machines-in-a-folder/

## Desactivation du vérouillage automatique de la session windows à la fermeture de la console

Modifiez la variable : **tools.guest.desktop.autolock** à FALSE

Via powerCLI :
```powershell
GetVM -Name <NOM_DE_LA_VM> | New-AdvancedSetting -Name "tools.guest.desktop.autolock" -Value "false" -Confirm:$false -Force
```
