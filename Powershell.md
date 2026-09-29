# TP3 - Powershell niveau 2
## Script 1 (Recherche de dossier)
```powershell
<#------------------------------------------------------------------
Apprentissage PowerShell - Script n° 1
Fonction : Ce script cherche un fichier dans un dossier donné
Auteur YH – 28/09/2026
--------------------------------------------------------------------#>
$cherche = $args[0]
$dossier = $args[1]
$nbfichier = 0
Write-Output "Recherche du fichier $cherche dans le dossier $dossier"
Get-ChildItem -Path $dossier -ErrorAction SilentlyContinue -Recurse | `
    Where-Object {$_.Name -eq $cherche} | `
    forEach-Object {
        Write-host ("le fichier $cherche est dans "+$_.DirectoryName)
        $nbfichier++
    }
Write-Host -foregroundcolor yellow "le fichier $cherche est présent dans $nbfichier dossiers" 
```
## Script 2 (Calcule la taille d'un dossier/fichier)
```powershell
<#-------------------------------------------------------------------
Apprentissage PowerShell - Script n° 2
Auteur YH – 28/09/2026
---------------------------------------------------------------------#>
$dossier = $args[0]
Write-Host "calcul en cours sur $dossier"
Get-ChildItem -Path $dossier -Recurse -Force -ErrorAction SilentlyContinue | `
    Where-Object {$_PsisContainer -ne 0} | `
    Where-Object {$_.Length -gt 1MB} | `
    Measure-Object -property Length -Sum | `
        ForEach-Object {
        $total = $_.sum / 1MB
        write-host -foregroundColor yellow ("le dossier "+$dossier+" contient {0:#,##0.0} MB" -f $total)
 }
```
## Script 3 (Choisie la couleur du texte)
```powershell
<#-------------------------------------------------------------------
Apprentissage PowerShell - Script n° 3
Auteur YH – 28/09/2026
---------------------------------------------------------------------#>
$listeCouleurs = @("Black","DarkBlue","DarkGreen","DarkCyan","DarkRed","DarkMagenta","DarkYellow","Gray","DarkGray","Blue","Green","Cyan","Red","Magenta","Yellow","White")
$couleur = ""
while ($couleur -ne 'stop') {
    $invite = "saisissez une couleur"
    $couleur = Read-Host $invite
    $z = $listeCouleurs | where-object {$_ -match $couleur}
    if ($z -ne $null) {
        Write-Host -ForegroundColor $couleur ("vous avez demandé à écrire en "+$couleur)
    }
    else {
        write-host ("la couleur "+$couleur+" n'existe pas.")
    }
}
```
## Script 4 (Crée un dossier ou un sous dossier)
```powershell
<#-------------------------------------------------------------------
Apprentissage PowerShell - Script sodecaf.ps1
Auteur YH – 28/09/2026
---------------------------------------------------------------------#>
ipcsv ".\utilisateurs sodecaf.csv" -Delimiter ";" | foreach {
    $triGramme=$_.firstname.substring(0,1)+$_.lastname.substring(0,1)
    $triGramme = $triGramme + $_.lastname.substring($_.lastname.length-1,1)
    $trigramme = $triGramme.toUpper()
   
    $agence = $_.agency
    # chemin absolu du dossier de chaque utilisateur
    $dossier = "D:\"+$agence+"\fic_"+$triGramme

    if ((Test-Path -Path ("D:\"+$agence)) -eq $false) {
        New-Item -Path "D:\" -Name $agence -ItemType "Directory"
    }
    
   if ((Test-Path -Path ($dossier)) -eq $false) {
        New-Item -Path $dossier -ItemType "Directory"
    }

    $couleur = switch ($_.function) {
        "informatique" {"cyan"}
        "comptable" {"yellow"}
        "accueil" {"blue"}
        Default {"white"}
    }
    Write-Host -ForegroundColor $couleur ($_.firstname+" "+$_.lastname+" ("+$triGramme+") "+$_.phone1)
}
```
## Script 5 (Crée un serveur DHCP avec une étendue)
```powershell
<#-------------------------------------------------------------------
Apprentissage PowerShell - Script tp4-DHCP.ps1
Fonction : installation et configuration du service DHCP
Auteur YH – 28/09/2026
---------------------------------------------------------------------#>

$NomServeur = "srv-win-core1.sodecaf.local"
$AdresseServeur = "172.16.0.2"
$NomEtendue = "DHCP_sodecaf"
$IPDebut = "172.16.0.150"
$IPFin = "172.16.0.200"
$Masque = "255.255.255.0"
$IPPasserelle = "172.16.0.254"
$DNSPrimaire = "172.16.0.1"
$DNSSecondaire = "1.1.1.1"
$DuréeDeBail = "14400"
$AdresseReseau = "172.16.0.0"

# Installation de la fonctionnalité DHCP sur le serveur
Install-WindowsFeature -Name DHCP -IncludeManagementTools

# Création d'un groupe de sécurité DHCP
Add-DhcpServerSecurityGroup

# Redémarrage du service DHCP
Restart-Service dhcpserver

# Autoriser le serveur DHCP dans l'annuaire
Add-DhcpServerInDC -DnsName $NomServeur -IPAddress $AdresseServeur

# Post-déploiement du service DHCP
Set-ItemProperty –Path registry::HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\ServerManager\Roles\12 –Name ConfigurationState –Value 2

# Effacer l'étendue si elle existe deja
if ((Get-DhcpServerv4Scope -ScopeId $AdresseReseau) -ne $null) {
    Remove-DhcpServerv4Scope -ScopeId $AdresseReseau -force
}

# Création d'une étendue
Add-DhcpServerv4Scope -Name $NomEtendue -StartRange $IPDebut -EndRange $IPFin -SubnetMask $Masque

# Ajout des options de l'etendue
Set-DhcpServerv4OptionDefinition -OptionId 6 -DefaultValue $IPPasserelle

# Ajout d'un DNS primaire, DNS Secondaire
Set-DhcpServerv4OptionDefinition -OptionId 6 -DefaultValue $DNSPrimaire,$DNSSecondaire

# Ajout de la durée du bail
Set-DhcpServerv4OptionValue -OptionId 51 -ScopeId $AdresseReseau -Value $DuréeDeBail

# Activation de l'étendue DHCP
Set-DhcpServerv4Scope -ScopeId $AdresseReseau -Name $NomEtendue -State Active

# Vérification des étendues configurées sur le serveur
Get-DhcpServerv4Scope -ScopeId $AdresseReseau
```
