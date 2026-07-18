# 03 — Remédiation (le cœur du dépôt)

> Si vous ne lisez qu'une page, c'est celle-ci. Objectif : rendre la chaîne décrite
> en §02 **inopérante**. Les contrôles sont classés par rapport coût/impact.

## §1 — Chiffrement de disque : LE contrôle qui casse la chaîne ⭐

**BitLocker (TPM + PIN de préboot idéalement).**

C'est le contrôle le plus rentable du dépôt. Il neutralise **l'étape A et l'étape B**
d'un coup : sans la clé, le disque système n'est ni lisible ni modifiable depuis
WinRE ou un média de boot. Toute la manipulation offline devient impossible.

- Déploiement piloté (Intune / GPO), séquestre des clés de récupération en AD/Entra.
- TPM seul protège contre le vol de disque ; **TPM + PIN** protège aussi contre
  l'attaquant qui a un accès console au poste allumé/éteint.
- Vérifier que WinRE est **aussi** sur volume chiffré ou désactivé (voir §4).

## §2 — Tamper Protection pilotée centralement, protégée par mot de passe

- Gérer la Tamper Protection **exclusivement depuis la console cloud** (Sophos
  Central, GravityZone, WatchGuard Cloud), pas en local.
- Exiger un **mot de passe / autorisation console** pour toute désinstallation.
- Alerter sur toute bascule d'un endpoint vers l'état « non protégé » ou
  « Tamper Protection désactivée ».

## §3 — Réduction du privilège admin local

L'ensemble de la chaîne **suppose l'admin local**. Le retirer élève massivement le coût.

- Retirer les utilisateurs des Administrateurs locaux (moindre privilège).
- **LAPS** (mot de passe admin local unique et rotatif par poste).
- Élévation à la demande contrôlée, journalisée.

## §4 — Verrouillage du boot et de la récupération

- **Secure Boot** activé, boot USB/PXE désactivé dans le firmware, **mot de passe BIOS/UEFI**.
- Restreindre / désactiver l'accès utilisateur à WinRE (`reagentc /disable` où c'est
  pertinent, ou WinRE sur volume protégé).
- Empêcher le déclenchement libre de `Redémarrage avancé` sur les postes sensibles.

## §5 — Détection et journalisation qui survivent au poste

- Remontée des logs vers un **SIEM externe** en quasi-temps réel : même si
  l'attaquant efface les traces locales (T1070), le SIEM a déjà l'événement.
- Surveiller la **santé des agents côté console** : un endpoint qui cesse de
  remonter « healthy » est un signal, pas un incident silencieux.
- Voir [`../detection/detections.md`](../detection/detections.md).

## §6 — Contrôles organisationnels

- Interdire l'usage de ces procédures hors process autorisé (déprovisioning outillé,
  journalisé, sur poste identifié).
- Utiliser les **outils de désinstallation officiels des éditeurs**, avec
  autorisation console, pour les agents légitimement orphelins.

## Matrice contrôle → étape neutralisée

| Contrôle | Étape A | Étape B | Étape C | Étape D | Variante |
|---|:---:|:---:|:---:|:---:|:---:|
| §1 Chiffrement disque | ✅ | ✅ | ⛔️ (indirect) | ⛔️ | ✅ |
| §2 Tamper central + MDP | | | ✅ | ✅ | |
| §3 Moindre privilège | ✅ | ✅ | ✅ | ✅ | ✅ |
| §4 Verrou boot/WinRE | ✅ | ✅ | | | ✅ |
| §5 Détection/SIEM | 🔎 | 🔎 | 🔎 | 🔎 | 🔎 |

✅ bloque · 🔎 détecte · ⛔️ empêche d'atteindre l'étape

## Priorisation express

1. **BitLocker (TPM+PIN)** — casse le vecteur offline, plus haut ROI.
2. **Moindre privilège + LAPS** — retire la précondition admin.
3. **Tamper Protection centrale + MDP** — verrouille le désarmement.
4. **SIEM externe + santé agents** — voir venir ce qui passe quand même.
5. **Verrou boot/WinRE** — ferme la porte de récupération.
