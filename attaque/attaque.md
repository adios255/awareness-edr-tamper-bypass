🛡️ Analyse d'une Procédure de Désinstallation Forcée de Sophos / Bitdefender

📅 Date de l'analyse : 22/11/2026
📄 Source : aebb8272-6fe6-49a1-9996-8c547a4d206d_Kill_Sophos__Bitdefender.pdf
🎯 Objectif : Comprendre, documenter et analyser les techniques utilisées pour désactiver et désinstaller des antivirus (Sophos et, potentiellement, Bitdefender) de manière non conventionnelle et forcée.
📌 1. Résumé Exécutif

Ce document décrit une procédure manuelle en plusieurs étapes pour désinstaller l'agent de sécurité Sophos d'une machine Windows. La mention de Bitdefender dans le titre suggère que la même approche ou un outil similaire (comme EndpointAgentTool.exe de WatchGuard) pourrait être adapté pour cibler Bitdefender, bien que la procédure détaillée ne concerne que Sophos.

La méthode est agressive et contourne les mécanismes de protection standard (comme la Tamper Protection) en modifiant directement le système d'exploitation et la base de registre.
🔧 Approche combinée :
Étape	Technique	Objectif
1️⃣	Contournement physique	Démarrage en mode sans échec pour empêcher les services Sophos de se lancer
2️⃣	Manipulation du noyau	Renommage du driver principal (SophosED.sys) pour le rendre inactif
3️⃣	Désactivation de la sécurité	Modification du registre pour désactiver la Tamper Protection
4️⃣	Exécution massive	Lancement de tous les désinstalleurs Sophos disponibles en mode silencieux

    💡 Note : La procédure mentionne également un outil nommé EndpointAgentTool.exe provenant d'un concurrent (WatchGuard) comme une alternative prometteuse, pouvant probablement cibler plusieurs éditeurs dont Bitdefender.

🔍 2. Analyse Détaillée de la Procédure
🧰 Étape 0 : Prérequis

Avant de commencer, l'attaquant doit disposer de :

    ✅ Droits Administrateur Locaux : Indispensable pour toute modification système.

    ✅ Accès Physique ou à Distance : Un accès au poste (via RDP, iLO, ou physiquement) est nécessaire pour redémarrer en mode avancé.

    ✅ Script de Désinstallation : Le fichier SophosUninstall.bat doit être présent sur le bureau de l'utilisateur ou à un emplacement connu.

🚀 Étape 1 & 2 : Accès au Mode de Récupération et à l'Invite de Commandes

Action : L'utilisateur redémarre la machine et accède aux options de récupération de Windows.
📋 Procédure :

    Ouvrir les Paramètres Windows

    Naviguer vers Système → Récupération → Démarrage avancé

    Redémarrer le poste en mode avancé

    Sélectionner Dépannage

    Aller dans Options avancées → Invite de commandes

🧠 Explication Technique :

    Pourquoi le Mode de Récupération ? En démarrant sur l'environnement de récupération (WinRE), l'attaquant peut ouvrir une invite de commandes avant que le système d'exploitation principal et ses services ne soient complètement chargés.

    L'objectif : Empêcher les services Sophos de se lancer. Si l'antivirus est actif, il verrouille ses fichiers et processus pour empêcher toute modification. En se plaçant en dehors de l'environnement Windows normal, l'attaquant peut interagir avec le système de fichiers sans être gêné par les "hooks" et la protection en temps réel de Sophos.

💾 Étape 3 : Identifier le lecteur système

Action : Dans l'invite de commandes, identifier la lettre du disque système.
📋 Procédure :
cmd

diskpart
list volume
exit

🧠 Explication Technique :

    diskpart est un outil de gestion des disques.

    list volume affiche toutes les partitions.

    L'attaquant identifie la lettre du lecteur système (souvent C:) car, dans l'environnement de récupération, la lettre du disque peut être différente (ex: D:).

    C'est une étape cruciale pour ne pas modifier le mauvais disque.

🧩 Étape 4 : Renommer le fichier du driver Sophos

Action : Navigation vers le dossier des drivers et renommage du fichier SophosED.sys.
📋 Procédure :
cmd

C:
cd Windows\System32\drivers
ren SophosED.sys SophosED.sys.old

🧠 Explication Technique :

    Qu'est-ce que SophosED.sys ? C'est le fichier driver (pilote) du noyau pour Sophos Endpoint Defense.

    Un driver fonctionne au niveau le plus bas du système (Ring 0) et est chargé au démarrage de Windows.

    L'objectif du renommage : En renommant le fichier, l'attaquant s'assure que le pilote ne sera pas chargé par le gestionnaire de sessions de Windows lors du prochain démarrage.

    Sans ce driver, la couche "kernel" de Sophos est neutralisée. Le système peut alors démarrer normalement sans que l'antivirus n'ait de contrôle en profondeur.

🧪 Étape 5 : Exécution du script de désinstallation

Action : Après avoir neutralisé le driver, on redémarre normalement et on exécute un script .bat en tant qu'Administrateur.
📋 Procédure :

    Taper exit pour fermer l'invite de commandes

    Sélectionner Continuer vers Windows

    Une fois sur le bureau, localiser le script SophosUninstall.bat

    Ouvrir une invite de commandes en mode administrateur

    Naviguer vers le dossier contenant le script :

cmd

cd C:\Users\rems.w11.test3\Desktop

    Exécuter :

cmd

.\UninstallSophos.bat

🔧 Analyse du Script

Le script effectue deux actions principales :
A. Désactivation des services et de la Tamper Protection (via le Registre)
batch

:: Désactiver le service MCS Agent
reg add "HKLM\SYSTEM\CurrentControlSet\Services\Sophos MCS Agent" /v Start /t REG_DWORD /d 4 /f

    Explication : La clé de registre Start définit comment le service est lancé. La valeur 4 signifie Désactivé. Le service Sophos MCS Agent ne démarrera donc plus.

batch

:: Désactiver la Tamper Protection pour les sous-services
reg add "HKLM\SYSTEM\CurrentControlSet\Services\Sophos Endpoint Defense\TamperProtection\Services\hmpalert" /v Protected /t REG_DWORD /d 0 /f

    Explication : La Tamper Protection est une fonction de sécurité qui protège les fichiers et processus de Sophos contre les altérations. Ici, l'attaquant modifie la clé Protected en la mettant à 0 (désactivée).

    Cela est fait pour plusieurs services :

        hmpalert (HitmanPro.Alert)

        Sophos System Protection Service

        sophosztnatap

        et la clé Config avec SEDEnabled = 0

    Résultat : En désactivant la Tamper Protection, l'attaquant peut désinstaller les logiciels sans que Sophos ne bloque l'opération ou ne remette le système en état.

B. Lancement des désinstalleurs officiels
batch

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

    Explication : Le script exécute les désinstalleurs officiels de tous les composants Sophos avec l'argument -y (pour "yes", mode silencieux).

    Il force l'exécution des désinstalleurs un par un pour chaque composant : Agent, Endpoint Defense, ML Engine, UI, Firewall, HitmanPro.Alert, etc.

🔄 Pourquoi exécuter le script deux fois ?

    "je relance le script une 2e fois pour lancer la désinstallation."

Il est mentionné que le script est relancé une seconde fois. Cela permet de s'assurer que tous les composants sont bien pris en compte, surtout si la Tamper Protection n'était pas complètement désactivée lors du premier passage.
📊 Taux de réussite :

    "Taux de réussite à 99% (VM AD capout lol)"

🧰 Étape 6 : Finalisation

Action : Redémarrer l'ordinateur pour finaliser la désinstallation.
cmd

shutdown /r /t 0

    ⚠️ Important : Cette procédure nécessite des droits administrateur et doit être effectuée avec précaution et éthique.

🧪 3. La Technique "WatchGuard" (Mentionnée en Page 6)

Le document mentionne un outil externe : EndpointAgentTool.exe provenant de WatchGuard.
📋 Contexte :

    "Découverte d'un outil capable de supprimer l'agent, phase de test 20/11/2025"
    "EndpointAgentTool.exe"
    "source : https://www.watchguard.com/help/docs/help-center/fr-fr/content/en-us/endpoint-security/installation/create-template-vdi-persistent.html"

🧠 Analyse :

    Contexte légitime : Cet outil est normalement conçu pour désinstaller les agents de sécurité précédemment installés sur un poste avant de déployer la solution WatchGuard. Il est utilisé dans des environnements VDI (Virtual Desktop Infrastructure).

    Le détournement : L'attaquant a détourné l'utilisation de cet outil légitime pour qu'il cible Sophos (et probablement Bitdefender). Cela suggère que l'outil de WatchGuard est capable de nettoyer des traces d'autres antivirus (un cas courant pour les solutions de sécurité concurrentes).

    Conclusion : C'est une technique plus automatisée et probablement plus fiable (taux de réussite mentionné à 99%).

🤔 Et Bitdefender dans tout ça ?

La présence de Bitdefender dans le titre peut s'expliquer de plusieurs manières :

    L'outil WatchGuard est également efficace contre Bitdefender

    Une deuxième phase de test (non documentée) vise Bitdefender

    La procédure Sophos est adaptable à Bitdefender avec quelques modifications (renommer bd*.sys, désactiver les services Bitdefender, etc.)

🔗 4. Synthèse de la Chaîne d'Attaque

graph TD
    A[Accès Administrateur] --> B[Démarrage en Mode Récupération]
    B --> C[Identification du disque système]
    C --> D[Renommage du driver: SophosED.sys → .old]
    D --> E[Redémarrage normal]
    E --> F[Exécution du script en Admin]
    F --> G[Désactivation Tamper Protection via Registre]
    G --> H[Lancement des désinstalleurs officiels -y]
    H --> I[Second passage du script]
    I --> J[Redémarrage final]
    J --> K[✅ Sophos désinstallé]

🛡️ 5. Implications Sécurité et Défenses

Cette procédure est une attaque avancée. Elle suppose que l'attaquant a déjà des privilèges administratifs sur la machine.
🚨 Comment se défendre ?
Mécanisme de défense	Description
Restreindre les droits Administrateurs	La première ligne de défense. Si l'utilisateur n'est pas admin, il ne peut pas renommer les drivers système.
BitLocker et Secure Boot	Empêcher le démarrage sur des périphériques externes ou le mode de récupération sans authentification.
Surveillance des logs Windows (Event 4657)	Surveiller les modifications de la clé de registre TamperProtection.
Surveillance des fichiers système (Sysmon)	Alerter sur le renommage des fichiers .sys dans System32\drivers.
EDR (Endpoint Detection and Response)	Un bon EDR détectera une chaîne d'actions suspectes (démarrage en mode sans échec + modifications de registre + arrêts de services) plutôt que des signatures spécifiques.
Politique de sécurisation du WinRE	Restreindre l'accès au mode de récupération ou exiger un mot de passe.
📝 6. Conclusion

Cette procédure est une "recette de cuisine" efficace pour contourner la protection de Sophos (et potentiellement Bitdefender). Elle est dangereuse car elle manipule le noyau et désactive des mécanismes de sécurité critiques.

Le fait qu'une solution concurrente (WatchGuard) soit utilisée comme outil de désinstallation montre que ces techniques sont parfois facilitées par des outils légitimes.
📊 Points clés à retenir :

    ✅ La procédure est manuelle et complexe (nécessite 6 étapes)

    ✅ Elle contourne la Tamper Protection par le registre

    ✅ Elle neutralise le driver kernel de Sophos

    ✅ L'outil WatchGuard offre une alternative automatisée

    ⚠️ La mention de Bitdefender suggère une adaptabilité de la méthode

⚖️ Avertissement Éthique et Légal

    ⛔ Cette procédure est strictement documentée à des fins de recherche et d'éducation en cybersécurité.

    Toute utilisation sur un système dont vous n'êtes pas le propriétaire ou sans autorisation explicite est ILLÉGALE.

    L'utiliser sur un poste de travail d'une entreprise dans le but de contourner la politique de sécurité expose à :

        Des poursuites judiciaires

        Un licenciement immédiat

        Des dommages et intérêts

    Restez éthiques, restez légaux.

📚 Références

    Documentation WatchGuard sur EndpointAgentTool.exe : Lien

    Documentation Microsoft sur le mode de récupération : Lien

    Base de registre Windows : Services et paramètres de démarrage

Document analysé et structuré pour une meilleure compréhension des techniques d'évasion antivirale.
