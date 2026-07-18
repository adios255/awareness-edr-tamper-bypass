# 01 — Contexte

## Pourquoi ce support

Les EDR modernes (Sophos Intercept X, Bitdefender GravityZone, WatchGuard EPDR…)
embarquent une **Tamper Protection** : un mécanisme qui empêche un utilisateur —
même administrateur local — d'arrêter les services, de supprimer l'agent ou de
modifier sa configuration depuis le système en fonctionnement.

Le message que beaucoup d'admins retiennent à tort : *« avec la Tamper Protection,
mon EDR est indéboulonnable »*. Ce support existe pour corriger cette croyance et
montrer **où se situe réellement la limite de confiance** — puis comment la déplacer
au bon endroit.

## À qui ça s'adresse

- Équipes IT / MSP qui déploient et administrent des EDR en parc.
- RSSI et responsables sécurité qui construisent une politique de durcissement.
- Auditeurs et blue teams qui veulent des règles de détection prêtes à l'emploi.

## Périmètre — ce que ce dépôt fait et ne fait pas

**Fait :**
- Décrit la logique d'une chaîne de contournement, au niveau TTP.
- Mappe chaque étape sur MITRE ATT&CK.
- Fournit détection + remédiation actionnables.

**Ne fait pas :**
- Ne fournit **aucun script ni commande copiable** permettant de désarmer un EDR.
- N'est pas un mode opératoire d'attaque.

Ce choix est assumé : pour sensibiliser, on n'a pas besoin de livrer l'arme. On a
besoin de montrer la faille de conception (confiance excessive dans un contrôle
software qui suppose l'intégrité de l'OS) et de fermer la porte.

## Le principe de fond

Toute la chaîne repose sur **une seule hypothèse cassable** : l'attaquant peut
lire/écrire le disque système **hors du contrôle de l'OS protégé** (via
l'environnement de récupération, un boot alternatif, ou un accès offline). Dès que
cette hypothèse tombe — typiquement grâce au **chiffrement de disque** — la majorité
de la chaîne devient inopérante. C'est le fil rouge de tout le dépôt.
