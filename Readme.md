# awareness-edr-tamper-bypass

> **Support de sensibilisation défensif.** Ce dépôt documente une chaîne d'attaque
> réelle contre les protections endpoint (EDR/AV) afin d'aider les équipes IT et
> sécurité à **détecter et bloquer** ce type de contournement. Il ne contient
> **aucun script offensif exécutable**.

## ⚠️ Avertissement

Ce contenu est fourni à des fins **éducatives et défensives** uniquement. Il décrit
une TTP (technique d'attaquant) au niveau conceptuel, mappée sur MITRE ATT&CK, pour
que les défenseurs comprennent le risque et le neutralisent. Aucune commande
copiable permettant d'armer l'attaque n'est publiée ici — c'est un choix délibéré.

## Le message en une phrase

> **Un attaquant qui obtient les droits admin local + un accès à l'environnement de
> récupération (WinRE) peut neutraliser un EDR pourtant équipé de Tamper Protection.**
> Le contrôle qui casse toute la chaîne est simple : **le chiffrement de disque (BitLocker).**

## Ce que ça démontre

- La Tamper Protection d'un EDR est une **barrière, pas un mur**. Elle protège contre
  la désactivation *depuis l'OS en ligne*, pas contre une manipulation *offline*.
- L'accès physique / recovery + admin local reste un vecteur sous-estimé.
- La défense la plus rentable n'est pas « un meilleur agent », c'est une combinaison
  de contrôles de configuration (voir [`docs/03-remediation.md`](docs/03-remediation.md)).

## Sommaire

| Fichier | Contenu |
|---|---|
| [`docs/01-contexte.md`](docs/01-contexte.md) | Pourquoi ce sujet, périmètre, public visé |
| [`docs/02-chaine-attaque.md`](docs/02-chaine-attaque.md) | La chaîne décrite (conceptuelle, mappée ATT&CK) |
| [`docs/03-remediation.md`](docs/03-remediation.md) | **Le cœur du dépôt** : durcissement |
| [`detection/detections.md`](detection/detections.md) | Règles de détection (SIEM / EDR) |
| [`attaque/attaque.md`](attaque/attaque.md) | Analyse d'une Procédure de Désinstallation Forcée de Sophos / Bitdefender |

## Licence

Documentation sous licence éducative. Voir la section remédiation avant toute chose.
