# 02 — La chaîne d'attaque (description conceptuelle)

> Cette page décrit **ce que fait** un attaquant et **pourquoi ça marche**, sans
> fournir les commandes. L'objectif est que le défenseur reconnaisse la technique
> et sache où placer un contrôle. Chaque étape renvoie à la remédiation qui la casse.

Technique principale : **MITRE ATT&CK [T1562.001 — Impair Defenses: Disable or Modify Tools](https://attack.mitre.org/techniques/T1562/001/)**

## Vue d'ensemble

```
[Admin local obtenu]
        │
        ▼
[Étape A] Accès à l'environnement de récupération (WinRE)  ── T1562.001 / T1553
        │   → manipulation OFFLINE du disque, hors OS protégé
        ▼
[Étape B] Neutralisation du driver de protection            ── T1562.001
        │   → le composant kernel qui applique la Tamper Protection
        │     ne se charge plus au prochain boot
        ▼
[Étape C] Désactivation de la Tamper Protection             ── T1562.001
        │   → une fois le garde-fou kernel absent, la config
        │     de protection redevient modifiable
        ▼
[Étape D] Désinstallation « propre » des composants         ── T1562.001
        │
        ▼
[Variante] Compte admin caché + safe boot + nettoyage        ── T1136.001 / T1562.009 / T1070
            → persistance temporaire + évasion + effacement de traces
```

## Étape A — Accès offline via l'environnement de récupération

**Idée :** en démarrant sur WinRE (ou tout média de boot alternatif), l'attaquant
manipule le système de fichiers **sans que l'EDR soit chargé**. La Tamper Protection
ne peut pas défendre ce qui ne tourne pas.

- ATT&CK : T1562.001, apparenté à T1553 (contournement de contrôles).
- **Cassé par :** chiffrement de disque (le volume est illisible/immodifiable sans
  la clé), verrouillage du boot WinRE/USB. → voir remédiation §1 et §4.

## Étape B — Neutralisation du composant kernel de protection

**Idée :** le cœur de la Tamper Protection est un driver noyau. S'il ne se charge
pas au démarrage, la protection n'est plus appliquée. Offline, un attaquant peut
empêcher ce chargement.

- ATT&CK : T1562.001.
- **Cassé par :** intégrité du volume (chiffrement), et détection au reboot d'un
  agent qui ne remonte plus « healthy » côté console. → remédiation §1, détection §B.

## Étape C — Désactivation de la Tamper Protection

**Idée :** une fois le garde-fou kernel neutralisé, les réglages qui protègent
l'agent (les indicateurs « protégé » des différents services, l'interrupteur global)
redeviennent modifiables. C'est l'inversion du contrôle censé être inviolable.

- ATT&CK : T1562.001.
- **Cassé par :** Tamper Protection **pilotée par mot de passe depuis la console
  centrale** (et non modifiable localement), alerting sur toute modification de la
  configuration de protection. → remédiation §2, détection §C.

## Étape D — Désinstallation des composants

**Idée :** l'agent étant désarmé, les désinstalleurs officiels s'exécutent sans
opposition.

- ATT&CK : T1562.001.
- **Cassé par :** tout ce qui précède empêche d'arriver ici ; en défense en
  profondeur, alerting console sur passage d'un endpoint à l'état « non protégé ».

## Variante Bitdefender / safe boot

**Idée :** création d'un **compte administrateur local caché**, bascule en
**safe boot** (où beaucoup de protections ne se chargent pas), exécution de l'outil
de suppression, puis **suppression du compte et effacement des fichiers** pour
brouiller les traces.

- ATT&CK : T1136.001 (Create Account), T1562.009 (Safe Mode Boot),
  T1070 (Indicator Removal).
- **Cassé par :** interdiction/alerting de `bcdedit safeboot`, détection de création
  de comptes admin hors IAM, journalisation centralisée qui survit à l'effacement
  local. → détection §A et §D.

## À retenir

La chaîne n'exploite **aucune vulnérabilité mémoire ni 0-day**. Elle exploite une
**hypothèse de conception** : « l'attaquant n'a pas d'accès offline au disque ».
Retirez cette hypothèse, et la chaîne s'effondre. C'est tout l'objet de la page
suivante.
