# Détections (SIEM / EDR)

> Règles orientées **comportement**, pas signature. Elles ciblent les effets
> observables de la chaîne §02, indépendamment du script précis utilisé.
> Adaptez les noms de champs à votre stack (Sentinel, Splunk, Elastic, XDR natif).

## §A — Bascule en Safe Boot & comptes admin suspects

**Safe boot forcé** (T1562.009) — signal fort, très rare en usage légitime :
- Détecter l'exécution de `bcdedit` modifiant la configuration `safeboot`
  (`process_creation` où `image` = `bcdedit.exe` et la ligne de commande référence
  `safeboot`). Alerte **haute** sur un poste utilisateur.

**Création de compte admin hors IAM** (T1136.001) :
- Windows **4720** (compte créé) suivi de **4732** (ajout au groupe Administrateurs),
  corrélés hors de votre process de provisioning autorisé.
- Bonus : compte créé **puis supprimé** (**4726**) dans une courte fenêtre = fort
  indicateur de nettoyage de traces.

## §B — Atteinte au driver / à la santé de l'agent

- **Endpoint qui cesse de remonter `healthy`** côté console EDR, surtout juste après
  un reboot : à traiter comme un événement, pas un silence.
- Absence de démarrage attendu d'un service/driver de protection au boot
  (comparer l'inventaire des services de protection attendus vs présents).
- Modification hors ligne détectable au retour en ligne : version d'agent qui
  régresse ou composants manquants.

## §C — Modification de la configuration de protection

- Toute écriture sur les clés de configuration de la **Tamper Protection** de l'EDR
  (surveillance registre / intégrité de configuration), en particulier un
  indicateur de protection passant à « désactivé » ou l'interrupteur global.
- Côté console : événement « Tamper Protection désactivée » ou « désinstallation
  initiée » sans ticket / autorisation associée.

## §D — Effacement de traces & manipulations WinRE

- Suppression d'outils/fichiers juste après exécution (T1070) — corréler
  création → exécution → suppression du même artefact dans une courte fenêtre.
- Déclenchement de **Redémarrage avancé / WinRE** sur poste sensible hors maintenance.
- Effacement de journaux d'événements (**1102** — Security log cleared).

## Logique de corrélation recommandée

Un seul de ces signaux = à investiguer. **Deux ou plus dans une courte fenêtre sur
le même hôte** (ex. `bcdedit safeboot` + création compte admin + agent unhealthy)
= **incident probable de défense-évasion** → réponse immédiate.

## Rappel

Ces détections supposent une **journalisation centralisée** (§5 de la remédiation).
Si les logs ne quittent jamais le poste, l'attaquant les efface avant que vous ne
les voyiez.
