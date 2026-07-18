# Kill Sophos / Bitdefender

> **Analyse d'une procédure de désinstallation forcée d'agents de sécurité**

| Métadonnée | Information |
| :--- | :--- |
| **📅 Date de découverte** | 01/01/2026 |
| **📢 Statut** | Éditeurs informés (Responsible Disclosure) |
| **🎯 Cibles** | Sophos Endpoint Defense / Bitdefender Endpoint Security |
| **📄 Source** | `aebb8272-6fe6-49a1-9996-8c547a4d206d_Kill_Sophos__Bitdefender.pdf` |

---

> [!CAUTION]
> **⚠️ AVERTISSEMENT LÉGAL ET ÉTHIQUE**
> Cette documentation est fournie **uniquement à des fins de recherche et d'éducation** en cybersécurité. L'utilisation de ces techniques sur des systèmes dont vous n'êtes pas le propriétaire ou sans autorisation explicite est **ILLÉGALE**. Toute utilisation malveillante expose à des poursuites judiciaires et à un licenciement immédiat. Les éditeurs ont été prévenus afin de corriger ces failles.

---

## 1. Contexte et Résumé

Le document analysé décrit une méthode de contournement "physique" permettant de désinstaller **Sophos** (et potentiellement **Bitdefender**) même lorsque la fonction de **Tamper Protection** (anti-altération) est activée.

La méthode ne consiste pas à exploiter une vulnérabilité logicielle classique, mais à **abuser des mécanismes de récupération de Windows** pour neutraliser manuellement le driver noyau de l'antivirus avant de désactiver ses verrous dans la base de registre.

> [!IMPORTANT]
> **Distinction Sophos / Bitdefender :**
> - La procédure détaillée pas-à-pas (Étapes 1 à 6) concerne **exclusivement Sophos**.
> - **Bitdefender** est mentionné dans le titre du document car l'outil alternatif découvert (`WatchGuard EndpointAgentTool`) est capable de nettoyer les traces de plusieurs éditeurs, et la technique est structurellement adaptable à Bitdefender (renommage des drivers `bd*.sys` et modification des services `Bitdefender`).

---

## 2. Procédure détaillée pour Sophos

Cette section détaille les 6 étapes de la procédure, avec les explications techniques du "pourquoi" chaque action est réalisée.

### 2.1. Prérequis
Avant de commencer, l'attaquant doit disposer de :
- **Droits Administrateur locaux** sur la machine.
- **Accès physique ou console** (RDP, iLO, IPMI) pour redémarrer en mode avancé.
- Le script `SophosUninstall.bat` présent sur le bureau ou accessible via le réseau.

---

### Étape 1 & 2 : Accès au Mode de Récupération et à l'Invite de Commandes

**Action :** L'utilisateur force le redémarrage en mode de récupération Windows (WinRE) et ouvre une invite de commandes.

**Procédure :**
1. Paramètres Windows → Système → Récupération → Démarrage avancé.
2. Redémarrer.
3. Sélectionner **Dépannage** → **Options avancées** → **Invite de commandes**.

> [!NOTE]
> **Explication technique :**
> En démarrant sur l'environnement de récupération, l'attaquant se place **en dehors** de l'installation Windows principale. Ainsi, les services et processus Sophos (qui tournent en Ring 0 / noyau) ne sont **pas chargés**. Cela permet de manipuler les fichiers système (comme les drivers `.sys`) sans être bloqué par la protection en temps réel de l'antivirus.

---

### Étape 3 : Identifier le lecteur système

**Action :** Trouver la lettre du disque système, car sous WinRE, la lettre peut être différente (souvent `D:` au lieu de `C:`).

**Procédure :**
```cmd
diskpart
list volume
exit
