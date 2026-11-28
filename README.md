# Planning de l'élevage

> Planning mensuel de l'élevage et du magasin, calculé au plus près des heures de chaque contrat, sans qu'aucune donnée ne quitte l'ordinateur.

Une page web autonome (`planning-elevage.html`), ouverte dans Google Chrome, qui répartit les créneaux du matin, du soir et du magasin entre les personnes de l'équipe, signale ce qui ne peut pas être tenu et laisse le gérant trancher. Elle fonctionne sans installation, sans serveur et sans connexion à Internet.

Ce document s'adresse d'abord à qui doit **vérifier l'outil** : un service technique qui contrôle la sécurité, ou un gérant, un comptable, un inspecteur qui doit comprendre **comment les heures sont comptées**. La prise en main vient ensuite.

![HTML](https://img.shields.io/badge/HTML%20%2B%20JavaScript-un%20seul%20fichier-3776AB)
![Dépendances](https://img.shields.io/badge/d%C3%A9pendances-aucune-16a34a)
![Réseau](https://img.shields.io/badge/connexion%20r%C3%A9seau-aucune-16a34a)
![Navigateur](https://img.shields.io/badge/navigateur-Google%20Chrome-4285F4)

## Sommaire

1. [Sécurité et confidentialité des données](#1-sécurité-et-confidentialité-des-données)
2. [Méthodes de calcul](#2-méthodes-de-calcul)
3. [Vérifier les calculs soi-même](#3-vérifier-les-calculs-soi-même)
4. [Mise en place](#4-mise-en-place)
5. [Utilisation](#5-utilisation)
6. [Fichiers, structure et modification de l'outil](#6-fichiers-structure-et-modification-de-loutil)

---

## 1. Sécurité et confidentialité des données

### 1.1 Données traitées

| Donnée | Où elle se trouve |
|---|---|
| Adresses e-mail des personnes (facultatives), proposées comme destinataires de l'envoi du planning | `planning-elevage-donnees.json` ; jamais écrites dans le journal |
| Prénoms et noms des personnes de l'équipe, heures du contrat, jours de repos, créneaux possibles | `planning-elevage-donnees.json`, dans le dossier choisi |
| Jours de congé, solde de congés au 1er juin | même fichier |
| Plannings enregistrés de chaque mois, mois validés et leurs écarts d'heures | même fichier |
| Historique des modifications (avec les prénoms et noms) | `planning-elevage.log`, même dossier |
| Lien vers le dossier choisi (objet opaque fourni par Chrome) | IndexedDB du navigateur, rien d'autre |

**Le dépôt ne contient aucune donnée personnelle ni propre à une exploitation.** La page est livrée vierge : le fichier de données est créé sans aucun créneau ni aucune personne. Le gérant saisit lui-même ses créneaux de travail (onglet **Réglages**), puis son équipe (onglet **Équipe**). Les deux fichiers de données sont exclus du dépôt par `.gitignore`, comme les sauvegardes téléchargées.

### 1.2 Ce que la page garantit

| Garantie | Comment elle est obtenue |
|---|---|
| **Aucune donnée n'est envoyée sur Internet** | La page s'interdit elle-même toute connexion réseau (`connect-src 'none'` dans sa politique `Content-Security-Policy`). Même un code défectueux ne pourrait pas transmettre de données : le navigateur bloque la requête. |
| **Un seul lien vers l'extérieur, sans planning** | « Envoyer par Gmail » télécharge le PDF du planning, fabriqué dans la page (`planningPdf()`), puis ouvre Gmail dans le navigateur sur un message prérempli (`gmailUrl()`). Le lien ne s'ouvre que sur un clic du gérant, dans un nouvel onglet, et ne contient que des textes fixes, le nom du mois et les **adresses e-mail des destinataires** saisies dans l'onglet Équipe : ni nom, ni heure, ni planning. Le PDF est joint par le gérant lui-même. |
| **Aucun code extérieur n'est chargé** | Ni bibliothèque, ni police, ni image, ni outil de mesure d'audience (`default-src 'none'`). Tout ce qui s'exécute est écrit dans le fichier, et peut être relu. `eval()` reste interdit. |
| **Aucun serveur, aucun compte, aucun mot de passe** | Il n'y a rien à pirater en ligne : les données sont deux fichiers dans un dossier de l'ordinateur. |
| **Accès limité à un seul dossier** | La page ne lit et n'écrit que deux fichiers aux noms fixés par le code, dans le seul dossier que l'utilisateur a choisi et autorisé dans Chrome. Cette autorisation se retire à tout moment dans le navigateur. |
| **Rien de sensible dans le navigateur** | Chrome ne garde que le lien vers le dossier. Aucun nom, aucune heure, aucun planning n'y est stocké. |
| **Aucun fichier écrasé** | Un fichier de données existant n'est jamais recréé. S'il est illisible, la page refuse de l'ouvrir et ne modifie rien. |
| **Fichier modifié à la main contrôlé** | À l'ouverture, chaque valeur est vérifiée et bornée : un champ inconnu est ignoré, une valeur hors limites est remplacée par la valeur par défaut. |
| **Double contrôle des saisies** | Chaque saisie est contrôlée à l'écran, puis contrôlée de nouveau au moment de l'appliquer aux données. Si une seule valeur est invalide, rien n'est enregistré. |
| **Texte saisi neutralisé** | Les noms, le titre et les noms de créneaux sont échappés avant affichage : un nom contenant du code ne peut pas s'exécuter. Dans le journal, ils sont nettoyés : un nom ne peut pas créer une fausse ligne. |
| **Aucune adresse de dossier sur les documents imprimés** | Le calendrier et les fiches s'impriment sans les en-têtes et pieds de page de Chrome : ni date, ni titre, ni adresse du fichier (qui révélerait l'emplacement du dossier et le nom de la session). |
| **Un journal sans auteur nominatif** | Chaque ligne indique le rôle (`Administration`), jamais le nom de la personne qui a agi ni l'ordinateur. |
| **Aucune modification perdue sans avertissement** | Si l'écriture échoue (disque débranché, par exemple), les modifications restent à l'écran, un lien « Réessayer d'enregistrer » apparaît, et Chrome demande confirmation avant de fermer la page. Il en va de même pour un brouillon ou des réglages non enregistrés. |

### 1.3 Ce que la page ne fait pas

Ces limites sont à connaître avant la mise en place.

- **L'interrupteur « Administration » n'est pas une protection.** Il n'a pas de mot de passe et sert seulement à masquer les onglets Équipe et Réglages. Toute personne qui ouvre la page peut l'activer.
- **Les données ne sont pas chiffrées.** `planning-elevage-donnees.json` et `planning-elevage.log` sont lisibles par toute personne ayant accès au dossier. Le journal contient les prénoms et noms des salariés.
- **Le contrôle d'accès repose sur l'ordinateur.** Qui peut ouvrir la session et le dossier peut modifier le planning, y compris en ouvrant le fichier JSON à la main.
- **Si le dossier est synchronisé vers un service en ligne** (OneDrive, iCloud Drive, Google Drive, Dropbox…), ce service copie les fichiers dans le cloud. Ce n'est pas la page qui les y envoie, mais le résultat est le même : à décider en connaissance de cause.
- **Un planning envoyé par Gmail quitte l'ordinateur.** Le PDF joint au message et le destinataire saisi passent par les serveurs de Google, comme pour tout courriel. La page n'envoie rien elle-même ; le PDF enregistré reste aussi dans le dossier Téléchargements.
- **Une sauvegarde téléchargée sort du dossier.** Elle arrive dans le dossier Téléchargements de l'ordinateur : à ranger ou à supprimer ensuite.
- **Un seul utilisateur à la fois.** Si la page est ouverte dans deux onglets ou sur deux ordinateurs en même temps, la dernière écriture remplace l'autre.

### 1.4 Recommandations

1. Protéger la session de l'ordinateur par un mot de passe, et activer le chiffrement du disque (BitLocker sous Windows, FileVault sous macOS) : les fichiers sont alors illisibles si l'ordinateur ou le disque est volé.
2. Ranger la page et ses données dans un dossier réservé, hors d'un espace synchronisé en ligne si les données ne doivent pas quitter l'ordinateur.
3. Avant un geste sans retour (retirer une personne, effacer un mois, modifier les créneaux), télécharger une copie des données : Réglages → « Télécharger une sauvegarde des données ». Ranger ces copies sur un support à part (clé USB, disque externe).
4. Garder Google Chrome à jour, et n'ouvrir la page que depuis son dossier.
5. Ne pas modifier le fichier JSON à la main : la page le contrôle, mais une valeur invalide serait remplacée par la valeur par défaut.
6. Les plannings et le journal contiennent des données personnelles de salariés : n'en conserver que ce qui est utile, et supprimer les sauvegardes devenues inutiles.

### 1.5 Auditer le code

Tout le code tient dans `planning-elevage.html`, lisible sans outil. Il est commenté pour une relecture par un service technique :

- un **guide de lecture** en tête de fichier : structure, flux de données, stockage, points de contrôle de sécurité, limites assumées, vocabulaire ;
- la **politique de sécurité** expliquée directive par directive ;
- chaque fonction décrite : rôle, paramètres, résultat, et points d'attention quand il y en a.

| Question d'audit | Où regarder |
|---|---|
| La page peut-elle émettre vers le réseau ? | Balise `Content-Security-Policy`, en tête de fichier |
| Quelle adresse externe la page peut-elle ouvrir ? | `gmailUrl()` et `startMail()` : seul lien externe, ouvert au clic ; il porte les adresses e-mail des destinataires, rien d'autre de personnel |
| Une donnée saisie peut-elle exécuter du code ? | `esc()` et la règle énoncée en tête du script Interface |
| Un nom peut-il fausser le journal ? | `Core.logLine()` et sa fonction `clean()` |
| Que vaut un fichier de données modifié à la main ? | `Core.normalize()` |
| Les saisies sont-elles contrôlées ? | Actions `save-team` et `save-all-settings`, puis `Core.applyOp()` |
| Quels fichiers sont lus et écrits ? | `fileIo()`, `openFolder()` |
| Le fichier neuf contient-il des noms ? | `openFolder()` et `Core.newData()` : ni créneau, ni personne |
| Un fichier d'une version antérieure est-il repris sans perte ? | `Core.fromLegacy()` (renommage des clés), puis `Core.normalize()` |
| Une modification peut-elle être perdue ? | `Core.Store` (`dirty`, `busy`, `flush`) et l'écouteur `beforeunload`, en fin de fichier |
| Que contient le journal ? | `Core.logLine()`, `record()` dans `Core.applyOp()` |

Contrôles rapides, depuis le dossier du projet :

```bash
# Aucune API réseau ni ressource externe dans le code : la commande ne doit rien afficher
grep -nE "fetch\(|new (XMLHttpRequest|WebSocket|EventSource)|sendBeacon\(|<(script|img|iframe|link)[^>]+(src|href)=" planning-elevage.html

# La politique de sécurité appliquée par le navigateur
grep -n 'http-equiv="Content-Security-Policy"' planning-elevage.html

# La seule adresse externe du code : le message Gmail prérempli (une seule ligne attendue)
grep -n "https://" planning-elevage.html
```

Dans Chrome, l'onglet **Réseau** des outils de développement reste vide pendant toute l'utilisation, et toute tentative de connexion apparaît dans la **Console** comme bloquée par la politique de sécurité.

---

## 2. Méthodes de calcul

Toutes les règles de calcul sont dans le module `Core` de `planning-elevage.html` (entre les repères `/*CORE-DEBUT*/` et `/*CORE-FIN*/`). Chaque règle ci-dessous indique la fonction qui l'applique.

### 2.1 Conventions

| Notion | Définition | Fonction |
|---|---|---|
| Jour | Date `AAAA-MM-JJ`. Calculs en temps universel : un changement d'heure ne décale jamais un jour | `parseISO()`, `addDays()` |
| Semaine | Du lundi au dimanche. Une semaine qui déborde sur le mois voisin ne compte que ses jours **dans** le mois | `weeksOfMonth()` |
| Jours de la semaine | Numérotés de 1 (lundi) à 7 (dimanche) | `weekday()` |
| Durée d'un créneau | Fin − début, en heures, selon les horaires de la saison du jour | `spanHours()`, `monthSlots()` |
| Saison | Été du dernier dimanche de mars (inclus) au dernier dimanche d'octobre (exclu), ou entre deux dates fixes au choix (Réglages) | `isSummer()` |
| Gérant | Personne cochée « Gérant » : sa présence est notée, **aucune heure n'est comptée**, il n'a ni objectif d'heures, ni repos imposé, ni congés déduits | `manager` dans les données |
| Activité | Texte libre saisi pour chaque créneau (par exemple « Élevage », « Magasin »). Un créneau n'a pas de nom : il est désigné par son activité et ses horaires d'hiver, « Élevage 9h–13h » (`shiftLabel()`). Les créneaux de même activité, sans tenir compte des majuscules, sont additionnés ensemble | `activitiesOf()` |
| Jours fériés | Non pris en compte | — |

### 2.2 Heures du mois d'un salarié

```
heures prévues   = somme des durées des créneaux où la personne est placée dans le mois
heures de congé  = jours de congé déduits × valeur d'un jour de congé            (voir 2.3)
heures à faire   = heures du contrat − heures de congé                           (au moins 0)
écart            = heures à faire − heures prévues
```

Fonction : `targetOf()`, utilisée à la fois par la génération (`propose()`) et par l'analyse (`analyze()`).

- **Les congés sont déduits des heures à faire ; ils ne s'ajoutent pas aux heures travaillées.** Les heures prévues ne contiennent donc que des heures réellement planifiées sur des créneaux.
- Les heures à faire ne dépendent que du contrat et des congés du mois : les heures sup. des mois précédents **n'y entrent pas** (voir 2.5).
- La génération vise toujours les heures à faire. Les heures sup. du mois sont **calculées automatiquement** : heures prévues − heures à faire (contrat moins congés). Elles se recalculent à chaque changement du planning, y compris fait à la main : placer quelqu'un en plus fait apparaître des heures sup.
- La carte de chaque salarié affiche « *heures prévues* sur *heures à faire* » et, en haut à droite, un badge `+Xh` (heures sup., vert-bleu) ou `−Xh` (heures en moins, orangé). La bulle du badge détaille le calcul et le cumul avec les mois validés. À côté, un second badge : « Congés restants : X » (solde de l'année de congés, orangé s'il est dépassé ; le détail du mois est dans la bulle). Les dimanches de repos manquants restent signalés dans « Points à vérifier ».
- La génération automatique vise exactement les heures à faire (`T` dans `propose()`), et l'analyse les compare aux heures prévues (`analyze()`) : les deux utilisent la même formule (`targetOf()`).
- Les heures sont additionnées telles quelles (créneau de 3 h = 3 h). Il n'y a ni majoration, ni pause déduite, ni heures supplémentaires calculées : l'outil planifie, il ne fait pas la paie.

### 2.3 Valeur d'un jour de congé

```
heures par semaine       = heures du contrat par mois × 12 ÷ 52
jours travaillés/semaine = 7 − jours de repos par semaine
valeur d'un jour         = heures par semaine ÷ jours travaillés par semaine, arrondi à l'heure
```

Fonction : `leaveDayHours()`.

| Contrat mensuel | Repos par semaine | Heures par semaine | Valeur exacte d'un jour | Valeur retenue |
|---|---|---|---|---|
| 151 h | 2 | 34,85 h | 6,97 h | **7 h** |
| 120 h | 2 | 27,69 h | 5,54 h | **6 h** |
| 104 h | 2 | 24,00 h | 4,80 h | **5 h** |
| 84 h | 2 | 19,38 h | 3,88 h | **4 h** |

**Plafond par semaine** (`weekRules()`) : une semaine ne peut pas déduire plus de jours de congé que la personne ne peut travailler cette semaine-là, soit les jours de la semaine dans le mois moins ses jours de repos. Un congé posé du lundi au dimanche (7 jours) avec 2 jours de repos déduit donc 5 jours : 5 × 7 h = 35 h pour un contrat de 151 h. Les 2 jours restants tombent sur les jours de repos et ne sont pas déduits ; la fiche individuelle l'indique.

**Dimanches et jours fériés : affichés, jamais décomptés** (`leaveCounts()`) : un congé posé un dimanche, ou un jour férié saisi dans Réglages, reste affiché au planning et sur la fiche d'heures (« dimanche : non décompté », « jour férié : non décompté »), mais il ne retire rien au solde de congés ni aux heures à faire. Il prend tout de même sa place dans le plafond de la semaine : une semaine entière de congé avec un lundi férié et 2 jours de repos déduit 4 jours (mardi à vendredi), pas 5. Un congé du jeudi au dimanche déduit 3 jours.

**Un jour de congé est un jour sans travail** : la personne n'est jamais placée ce jour-là, et le jour compte comme jour de repos, et comme dimanche de repos s'il tombe un dimanche.

**Effet de l'arrondi** : la valeur retenue peut s'écarter de la valeur exacte de moins d'une demi-heure par jour (par exemple + 0,12 h par jour pour 84 h, soit + 0,6 h pour une semaine entière de congé). L'arrondi à l'heure garde des heures à faire atteignables avec des créneaux en heures entières.

**Compteur de congés** (`leavePeriod()`, `leaveBalance()`) : l'année de congés va du 1er juin au 31 mai. Le solde au 1er juin se saisit à la main sur la fiche de chaque salarié (onglet Équipe, en jours, demi-journées acceptées).

```
jours posés      = jours de congé déduits dans l'année, comptés comme ci-dessus (au plus 7 − repos par semaine,
                   ni dimanche, ni jour férié)
solde restant    = solde au 1er juin − jours posés
```

Une semaine entière de congé avec 2 jours de repos compte donc 5 jours (jours ouvrés). Le solde restant s'affiche en mode « Congés », à chaque clic, sur la carte du salarié et sur sa fiche d'heures. Un solde négatif est signalé (« dépassé de 2 j ») mais n'empêche pas de poser le congé.

### 2.4 Repos et dimanches

| Règle | Calcul | Fonction |
|---|---|---|
| Repos hebdomadaire | Jours sans aucun créneau dans la semaine ≥ jours de repos du contrat (2 par défaut) | `weekRules()`, `cost()` |
| Semaine à cheval sur deux mois : repos | Si le planning du mois voisin est enregistré (généré ou saisi) : repos demandé − jours de repos déjà prévus dans ce mois-là pour cette semaine. Sinon : repos × jours de la semaine dans le mois ÷ 7, arrondi, et l'écran affiche un avertissement sur ces semaines | `weekRules()` |
| Semaine à cheval : heures | Repère d'heures de la semaine (sert à étaler les heures, l'objectif reste mensuel) : si le mois voisin est planifié, heures par semaine (contrat × 12 ÷ 52) − heures déjà prévues l'autre mois cette semaine − congés ; sinon prorata des jours | `weekRules()` |
| Semaine à cheval : congés | Les jours de congé sont comptés sur la semaine entière, au plus 7 − jours de repos, dans l'ordre des jours : chaque mois déduit les siens, sans double décompte | `weekRules()` |
| Dimanches espacés entre deux mois | Le dernier dimanche du mois précédent et le premier du mois suivant, s'ils sont planifiés, comptent pour « Dimanches de repos espacés » | `neighborSundays()`, `cost()`, `analyze()` |
| Dimanches de repos (règle de répartition, voir 2.7) | Dimanches sans aucun créneau dans le mois ≥ dimanches de repos demandés (2 par défaut, au plus le nombre de dimanches du mois) | `analyze()`, `cost()` |
| Durée d'une journée | Somme des créneaux d'un même jour pour un salarié : **10 h au plus**. C'est une constante du code (`MAX_DAY_HOURS`), pas un réglage. La génération ne la dépasse jamais (construction de départ et note : chaque heure au-delà coûte autant qu'un jour de repos manquant). Un dépassement fait à la main est signalé en rouge et demande confirmation à l'enregistrement. Le gérant n'est pas concerné | `propose()`, `cost()`, `analyze()` |
| Créneau indisponible (hors congés) | Créneau marqué « Indisponible » pour une personne un jour donné (rendez-vous médical…) : elle n'y est jamais placée, même en re-générant ; ses autres créneaux du jour restent possibles. **Contrairement à un congé, rien n'est décompté** : ni solde de congés, ni heures à faire (la génération reporte les heures sur d'autres créneaux, dans le respect des repos et des 10 h par jour). Un gérant indisponible sur un créneau ouvre le relais sur ce créneau | `canWork()`, `isBlocked()`, `isAway()` |

### 2.5 Compteur d'heures supplémentaires : mois validés

Un mois se **valide** avec le cadenas, à côté du choix du mois (`validateMonth` dans `Core.applyOp()`). À la validation, l'écart de chaque salarié est noté dans le fichier :

```
écart du mois           = heures prévues − (contrat − congés)                       (gardé dans l'historique)
heures sup. avant m     = somme des écarts des mois validés avant m                 (carryBefore())
heures à faire du mois  = contrat − congés − heures sup. avant m                    (targetOf(), au moins 0)
heures sup. cumulées    = heures sup. avant m + écart du mois m                     (0 si les heures à faire sont atteintes)
```

Exemple : compteur à −3 h fin janvier (heures dues). Février demande 152 h + 3 h = 155 h. 155 h prévues → écart de février +3 h, cumul 0 h. Inversement, +5 h à rendre → février demande 147 h. Si les règles (repos, créneaux cochés, 10 h par jour) empêchent d'atteindre ce total, le reste est reporté au mois suivant.

Le compteur est positif quand le salarié a fait des heures en plus (heures à lui rendre), négatif quand il en doit. **Il est purgé le mois suivant** : les heures à faire du mois en tiennent compte pour le ramener à 0 (`targetOf()`), et la génération vise ce total.

La carte de chaque salarié affiche « *heures prévues* sur *heures à faire* » (compteur compris) et les heures sup. du mois dans un badge (`+11h`) ; sous la barre, et dans la bulle, le cumul : « Cumul avec les mois validés : +8 h (à rendre) ». Pour un mois non validé, le cumul montre ce que deviendrait le compteur s'il était validé.

Un mois validé est verrouillé : ni génération, ni modification, ni effacement, ni congé (refusés aussi par `Core.applyOp()`). Le cadenas le déverrouille ; son écart sort alors du compteur jusqu'à une nouvelle validation. Les écarts notés ne changent pas si le contrat ou les créneaux changent ensuite. Seuls les mois validés comptent.

### 2.6 Repère hebdomadaire

Pour étaler les heures sur le mois, chaque semaine reçoit un repère :

```
repère de la semaine = heures du contrat × jours de la semaine dans le mois ÷ jours du mois − heures de congé de la semaine
```

Ce n'est pas un objectif : seul l'objectif mensuel (heures à faire) compte. Le repère n'entre pas dans la note : il sert seulement à choisir, pendant la construction de départ, qui compléter en premier.

### 2.7 Règles et priorités

| Priorité | Règle |
|---|---|
| Absolue | Chacun ne travaille que sur ses créneaux autorisés, les jours prévus pour lui |
| Absolue | Personne n'est sur deux créneaux qui se chevauchent |
| Absolue | Le gérant n'est jamais seul le matin (il ne compte pas dans le minimum de ce créneau) |
| Absolue | Personne n'est placé un jour de congé |
| Absolue | Jours de repos par semaine |
| 1 | Les créneaux indispensables sont tenus (ceux dont « Au moins » vaut 1 ou plus) |
| 2 et suivantes | Les **règles de répartition** cochées dans Réglages, dans l'ordre choisi par le gérant (voir ci-dessous) |
| Dernière | Les heures à faire du mois sont atteintes, le plus près possible de 0 h d'écart, en plus ou en moins |

**Règles de répartition** (Réglages → Règles de répartition, `RULES` et `ruleWeights()`). Chacune se coche ou se décoche ; leur ordre se change par glisser-déposer, ou avec les flèches. Ordre par défaut :

| Règle | Un manquement vaut |
|---|---|
| Dimanches de repos par mois (nombre réglé pour chaque salarié dans Équipe, 2 par défaut) | un dimanche de repos manquant |
| 2 jours de repos d'affilée, au moins une fois par mois (un jour de congé compte comme un repos) | un salarié qui n'en a jamais |
| Dimanches de repos espacés (éviter deux dimanches de repos de suite dans le mois) | une paire de dimanches de repos consécutifs |

La règle classée 1re pèse 10 000 000 000 par manquement, la 2e 1 000 000 000, et ainsi de suite (dix fois moins à chaque rang) ; une règle décochée ne pèse rien. Toutes passent après les créneaux indispensables (100 000 000 000 par heure) et avant les heures à faire (10 000 par heure d'écart). Elles sont signalées dans « Points à vérifier » quand elles ne sont pas tenues (seulement si elles sont cochées).

**Ajouter une règle** : une entrée dans `RULES` (identifiant, libellé, explication), son calcul dans `cost()` (`Core.propose()`) multiplié par son poids `W.<identifiant>`, et son alerte dans `analyze()`. Elle apparaît alors dans Réglages, cochée, à la fin de la liste.

Quand les règles absolues ou les règles de répartition empêchent d'atteindre les heures à faire, l'écart est affiché sur la carte de la personne et dans « Points à vérifier » : le gérant décide alors, à la main, d'une éventuelle exception. Les jours passés ne sont jamais modifiés par une génération, et les jours changés à la main sont gardés.

### 2.8 La note d'un planning

Chaque planning candidat reçoit une note (`cost()` dans `Core.propose()`). **Plus elle est basse, meilleur il est.**

```
note = 10 000 000 000 000 × personne sur deux créneaux en même temps
     +  1 000 000 000 000 × (gérant seul le matin + jours de repos hebdomadaire manquants)
     +    500 000 000 000 × créneaux « Ne jamais laisser vide » sans personne
     +    100 000 000 000 × heures de créneau indispensable sans personne
     + somme, pour chaque règle de répartition cochée, de  10^(10 − rang) × ses manquements   (voir 2.7)
       (ordre par défaut : 10 000 000 000 × dimanches de repos manquants
                         +  1 000 000 000 × salariés sans 2 jours de repos d'affilée
                         +    100 000 000 × paires de dimanches de repos consécutifs)
     +             20 000 × heures MANQUANTES par rapport aux heures à faire du mois
     +             10 000 × heures EN PLUS par rapport aux heures à faire du mois
     +       changeWeight × affectations changées par rapport au planning de référence (option reference de propose())
```

Les coefficients sont si éloignés qu'ils forment une hiérarchie : un créneau indispensable vide coûte plus cher que n'importe quelle règle de répartition, et une règle de répartition non tenue plus cher que n'importe quel écart d'heures. Une heure manquante compte double d'une heure en plus : à règles égales, la génération préfère dépasser un peu les heures à faire que les laisser incomplètes, pour **ne pas creuser les heures dues** par le salarié (entre manquer 2 h et dépasser de 3 h, elle dépasse). Les règles absolues et les règles de répartition passent toujours avant. Les jours de congé ne figurent pas dans la note : une personne en congé n'est jamais candidate ce jour-là, et ses heures de congé sont déjà retirées des heures à faire.

### 2.9 La recherche du meilleur planning

Le calcul construit plusieurs plannings et garde celui qui a la note la plus basse (`propose()`).

- **Construire** : les créneaux indispensables sont pourvus en premier, en commençant par ceux qui ont le moins de candidats. Les places restantes (deuxième personne le matin, magasin) servent ensuite à compléter les heures de ceux qui en manquent.
- **Améliorer** : trois gestes sont essayés partout, et gardés seulement s'ils font baisser la note. Une personne cède sa place à une autre. Deux places échangent leurs titulaires. Enfin, le « coup en deux temps » : un échange immédiatement complété par un ajout, par exemple le salarié A cède son dimanche matin au salarié B, reprend un matin de semaine et y ajoute le magasin.
- **Recommencer** : 150 essais au plus, en 2 secondes au plus. Un essai sur deux repart du meilleur planning trouvé, dont quelques jours sont vidés puis reconstruits.

Ce qu'il faut savoir sur le résultat :

- **Le hasard intervient.** Deux générations peuvent donner deux plannings différents, de qualité équivalente.
- **Le calcul cherche un très bon planning, pas le meilleur absolu garanti.** Sur une petite équipe, il l'atteint en pratique.
- **Congé posé après la génération** : la personne est retirée de ce jour dans le planning enregistré (`applyOp`, opération `leave`), sans autre changement. **Indisponibilité posée après la génération** : la personne est retirée de ce créneau (`applyOp`, opération `block`) ; le gérant choisit le remplaçant à la main. Sur un jour gardé à la main, la personne devenue absente sur un créneau est retirée et sa place recalculée ; le reste du jour gardé ne change pas.
- **Un total exact dépend de la durée des créneaux.** Avec des matins de 4 h et un magasin de 3 h, 151 h s'obtiennent par exemple avec 22 matins et 21 après-midi de magasin ; si les durées ne le permettent pas, la génération s'arrête à l'heure la plus proche.

### 2.10 Exemples chiffrés

Créneaux et équipe d'exemple, saisis à la main, horaires d'hiver :

| Activité | Jours | Hiver | Été | Personnes |
|---|---|---|---|---|
| Élevage (le matin) | tous | 9h–13h | 8h–12h | au moins 1, 3 au plus ; gérant jamais seul |
| Élevage (le soir) | tous | 17h–18h | 20h–21h | 1 |
| Magasin | tous | 14h30–17h30 | 14h30–17h30 | 0 à 1 (facultatif) |

| Personne | Contrat | Repos par semaine | Dimanches de repos | Peut travailler |
|---|---|---|---|---|
| Salarié A | 151 h | 2 jours | 2 | Matin ; magasin le lundi, mardi, jeudi, vendredi et samedi |
| Salarié B | 84 h | 2 jours | 2 | Matin |
| Gérant | présence seulement | — | — | Matin du lundi au samedi, jamais seul ; soir tous les jours |

| Mois | Salarié A | Salarié B | Remarque |
|---|---|---|---|
| Janvier 2027 | 148 h prévues sur 151 h à faire (88 h d'élevage, 60 h de magasin) : manque 3 h | **84 h** sur 84 h | Chacun garde ses 2 dimanches de repos. Pour 151 h, il faudrait qu'un des deux travaille un dimanche de plus |
| Février 2027 | 134 h sur 151 h : manque 17 h | 80 h sur 84 h : manque 4 h | 28 jours et 2 jours de repos par semaine : chacun ne peut travailler que 20 jours |
| Décembre 2026, A en congé du 14 au 20 | **116 h** prévues sur 116 h à faire (151 h − 35 h de congé : 5 jours × 7 h) | **84 h** | Pendant le congé de A, B est seul le matin mais garde ses 2 jours de repos : 2 matins restent sans personne, signalés au gérant |

L'écart vient chaque fois du repos hebdomadaire (règle absolue) ou des dimanches de repos (règle de répartition, ici cochée). Le gérant peut ajuster à la main : l'enregistrement lui signale l'exception et lui demande confirmation.

---

## 3. Vérifier les calculs soi-même

Le projet n'a pas de suite de tests automatisés : c'est un choix, pour un outil qui tient dans un seul fichier. La logique métier est isolée dans le module `Core`, qui ne touche ni à l'écran ni aux fichiers. Il s'extrait et s'exécute hors navigateur, par exemple sous macOS avec le moteur JavaScript du système :

```bash
sed -n '/\/\*CORE-DEBUT\*\//,/\/\*CORE-FIN\*\//p' planning-elevage.html > core.js
JSC=/System/Library/Frameworks/JavaScriptCore.framework/Versions/Current/Helpers/jsc
```

Nombre de créneaux de janvier 2027 avec les trois créneaux d'exemple (31 matins, 31 soirs, 31 après-midi de magasin) :

```bash
$JSC -e "load('core.js');
var d=Core.normalize({shifts:[
  {id:'m',activity:'Élevage',days:[1,2,3,4,5,6,7],winter:{from:'09:00',to:'13:00'}},
  {id:'s',activity:'Élevage',days:[1,2,3,4,5,6,7],winter:{from:'17:00',to:'18:00'}},
  {id:'b',activity:'Magasin',days:[1,2,3,4,5,6,7],winter:{from:'14:30',to:'17:30'}}]});
print(Core.monthSlots(d,'2027-01').length)"
# 93
```

Heures à faire d'un contrat de 151 h, en congé du 14 au 20 décembre 2026 :

```bash
$JSC -e "load('core.js');
var d=Core.normalize({employees:[{id:'a',name:'A',monthlyHours:151,restDaysPerWeek:2,sundaysOffPerMonth:2,
  leaveDays:['2026-12-14','2026-12-15','2026-12-16','2026-12-17','2026-12-18','2026-12-19','2026-12-20']}]});
var st=Core.analyze(d,'2026-12',{}).stats[0];
print(Core.leaveDayHours(st.person)+' h par jour, '+st.leaveCounted+' jours déduits, '+st.leaveHours+' h de congé, '+st.target+' h à faire')"
# 7 h par jour, 5 jours déduits, 35 h de congé, 116 h à faire
```

`Core.propose(données, {month:'2027-01'})` produit un planning, et `Core.analyze(données, '2027-01', planning.assignments)` en donne les heures et les alertes. Les chiffres de ce document ont été obtenus de cette façon, et l'interface a été testée dans Google Chrome.

---

## 4. Mise en place

Prérequis : Google Chrome (ou Microsoft Edge), sous Windows ou macOS.

1. Copier `planning-elevage.html` dans un dossier dédié sur le disque de l'ordinateur, par exemple `Documents/Planning élevage`.
2. L'ouvrir avec Chrome, cliquer sur « Choisir le dossier… » et sélectionner ce même dossier.
3. Accepter la création de `planning-elevage-donnees.json`. Cette étape n'a lieu qu'une fois. Le fichier est créé **vierge** : ni créneau, ni personne.
4. L'onglet **Réglages** s'ouvre : saisir le titre de la page (« Planning » par défaut ; un fichier qui porte encore l'ancien titre par défaut, « Planning de l’élevage », passe à « Planning »), cliquer sur « Ajouter un créneau » pour chaque créneau de travail et remplir la ligne (voir ci-dessous), vérifier l'ordre des règles de répartition, puis « Enregistrer les réglages ».
5. Onglet **Équipe** : saisir le prénom et le nom de chaque personne, y compris le gérant, une personne par ligne, puis « Ajouter ». Sur chaque fiche, régler les heures du contrat par mois, les jours de repos par semaine, les dimanches de repos par mois, et cocher les créneaux et les jours possibles ; cocher « Gérant » pour le gérant. Saisir le solde de congés au 1er juin. Cliquer sur « Enregistrer les modifications ».
6. Ouvrir **Planning**, choisir le mois et cliquer sur « Générer le planning ».

Chaque créneau se saisit dans des champs texte, sans liste déroulante :

| Champ | Contenu |
|---|---|
| Activité | Obligatoire. Texte libre, par exemple « Élevage » ou « Magasin ». Le créneau n'a pas de nom : il est désigné par son activité et ses horaires (« Élevage 9h–13h »). Deux créneaux de même activité doivent donc avoir des horaires d'hiver différents. Les heures sont additionnées par activité (carte « Temps par activité », fiche d'heures), et le planning individuel se filtre par activité |
| Jours | Les jours de la semaine où le créneau existe ; un nouveau créneau a tous les jours cochés, à décocher au besoin |
| Horaires d'hiver, d'été | **Facultatifs.** Début et fin au format HH:MM (« 9h », « 9h30 » ou « 9:30 » sont acceptés et remis en forme). Sans horaires pour une saison, le créneau n'existe pas pendant cette saison : il n'apparaît pas au planning. Un créneau ne passe pas minuit |
| Au moins | Personnes indispensables ; 0 = créneau facultatif |
| Au plus | Personnes au plus ; les places en plus servent à compléter les heures |
| Gérant jamais seul | Le gérant ne compte pas dans « Au moins » : il faut quelqu'un d'autre avec lui |
| Afficher si personne | « Afficher dans le planning même si personne n'est affecté ». Décoché, un créneau vide ce jour-là est masqué (un magasin sans personne de tout le mois disparaît du planning). Un créneau indispensable resté vide reste affiché, en rouge. Par défaut : coché si « Au moins » vaut 1 ou plus, décoché sinon |
| Ne jamais laisser vide | Une case par saison (été, hiver). **Relais du gérant** : un jour où le gérant est en congé, la liste de « Modifier » propose sur ce créneau tous les salariés de l'équipe (« relais gérant »), même s'il n'est pas coché sur leur fiche, sans alerte « pas prévu ». La génération, elle, ne les y place pas : elle ne suit que les créneaux cochés dans Équipe (les y cocher les ferait entrer dans la répartition de tous les jours). Le créneau doit toujours être tenu : quand le gérant est en congé, la génération y place un autre salarié en priorité (avant les autres créneaux indispensables) ; si c'est impossible, le créneau est affiché en rouge avec la raison et listé dans « Points à vérifier ». À la première ouverture, les cases sont pré-cochées d'après les horaires : en été les créneaux qui commencent à partir de 12 h (après-midi et soir), en hiver ceux qui commencent entre 12 h et 16 h 59 (après-midi) |

**Jours fériés** (carte « Jours fériés » de Réglages) : à ajouter à la main, autant que nécessaire (date et nom facultatif, par exemple « Noël »). Ils sont affichés au planning (« Férié · Noël »), sur la fiche d'heures et dans le PDF envoyé. Un congé posé ce jour-là n'est pas décompté (voir 2.3). Ils ne changent rien d'autre : les créneaux de ces jours-là restent à tenir et la génération les remplit comme les autres jours.

**Jours de présence du gérant** : ils se règlent sur **sa fiche, dans l'onglet Équipe** (créneaux et jours cochés), et nulle part ailleurs. Ces jours-là, le gérant est placé d'office (sauf en congé) ; la génération ne le place jamais ailleurs. Il peut toujours s'ajouter à la main dans le planning. Avec « Gérant jamais seul », « Au plus » doit valoir au moins 2. Une version précédente avait aussi une colonne « Gérant présent » dans Réglages, qui contredisait la fiche : si elle était remplie, ses jours sont repris sur la fiche du gérant à la première ouverture.

---

## 5. Utilisation

- **Planning** : le calendrier du mois, les cartes d'heures de chaque salarié, les points à vérifier et le temps par activité (une colonne par activité saisie).
  - Chaque bouton du planning s'explique **au survol** de la souris (bulle d'aide) : la page n'affiche pas ces explications en clair.
  - « Générer le planning » ou « Re-générer le planning » produit un **brouillon**. On le vérifie, on change une personne si besoin avec les listes, puis on clique sur « Enregistrer ».
  - « Modifier » ouvre un brouillon à partir du planning enregistré. Un jour changé à la main porte la mention « gardé » : une nouvelle génération **garde ses personnes**, mais peut **compléter ses places restées vides** (par exemple avec une disponibilité ajoutée ensuite dans Équipe). Un clic sur la mention libère le jour. Après une re-génération, le bandeau du brouillon indique le nombre de jours gardés, et « Tout recalculer (… jours gardés compris) » recalcule aussi ces jours (les changements faits à la main y sont alors remplacés). Les jours passés ne changent jamais.
  - **Disponibilité ajoutée après la génération** (un créneau coché en plus dans Équipe) : « Re-générer le planning » en tient compte sur tous les jours à venir, jours gardés compris (places vides).
  - « Congés » ouvre la saisie des congés : choisir la personne, puis cliquer sur les jours (un second clic retire le congé). Chaque clic est enregistré aussitôt et noté dans le journal ; le **solde de congés de l'année** (juin → mai) s'affiche et se met à jour à chaque clic. « Terminer » rend la main.
  - « **Indisponible** » marque les **créneaux** où une personne ne peut pas travailler **hors congés** (rendez-vous médical, par exemple). Choisir la personne : chaque jour du calendrier affiche ses créneaux prévus (onglet Équipe) sous forme de boutons ; un clic rend le créneau indisponible (barré), un second clic le rend disponible. Chaque clic est enregistré. Le jour porte ensuite « Indispo · Prénom : Élevage 9h–13h » (cadre en pointillés). La génération et les re-générations ne placent jamais la personne sur ces créneaux ; ses autres créneaux du jour restent possibles. **Rien n'est décompté**, ni des congés ni des heures à faire. Un congé posé le même jour remplace les créneaux indisponibles (il couvre la journée) : l'indisponibilité disparaît, et le congé est décompté des heures à faire comme tout congé. Si la personne était déjà placée sur le créneau dans le planning enregistré, elle en est retirée aussitôt : le gérant choisit qui la remplace avec « Modifier », sans re-générer.
  - **Congé posé après la génération** : la personne est **aussitôt retirée** de ses créneaux de ce jour-là dans le planning enregistré (noté au journal), et ses heures à faire baissent du congé. Le reste du planning ne bouge pas : pas besoin de re-générer. Une place indispensable laissée vide est signalée en rouge, à compléter avec « Modifier » si besoin.
  - **Indisponibilité posée après la génération** : la personne est **aussitôt retirée** de ce créneau dans le planning enregistré (noté au journal) ; le reste du planning ne bouge pas. Le gérant choisit lui-même qui la remplace avec « Modifier », sans re-générer (la place vide est signalée en rouge si elle est indispensable).
  - La carte de chaque salarié affiche en haut à droite ses **heures sup.** du mois dans un badge : `+Xh` (vert-bleu) ou `−Xh` (orangé), c'est-à-dire heures prévues − contrat moins congés, recalculées à chaque modification du planning. La bulle du badge donne le cumul avec les mois validés (voir 2.5), rappelé sous la barre : « Heures sup. : −8h ce mois · cumul −9h ». À côté : le badge « Congés restants : X ». Le nombre de jours travaillés est sur la fiche d'heures.
  - **Semaines à cheval sur deux mois** : si le mois voisin n'est pas encore planifié, un avertissement le signale, et la case hors du mois porte la mention « … pas encore planifié ». Re-générer le mois une fois le voisin planifié.
  - **Jours du mois voisin** (début et fin de calendrier) : ils montrent, grisés et pour information seulement, le planning enregistré de ce mois-là (créneaux et personnes, congés, créneaux indisponibles, jours fériés), filtré comme le reste dans le planning individuel. Rien n'y est cliquable : pour les modifier, ouvrir ce mois-là. Sans planning enregistré, seuls les créneaux du jour sont affichés.
  - **Planifier à l'avance** : tout mois peut être généré, modifié ou validé, même si le précédent n'est pas validé. L'historique des heures sup. ne compte que les mois validés et le mois en cours (`planBlocker()`) : un planning fait plus loin à l'avance n'y entre pas.
  - **Valider le mois** (cadenas ouvert, à côté du choix du mois) arrête les heures du mois : son écart s'ajoute au compteur d'heures sup. de chacun (voir 2.5), et le mois ne peut plus être modifié. Le cadenas fermé « Mois validé » le déverrouille.
  - « Imprimer le planning global », au-dessus du calendrier, produit le calendrier du mois de toute l'équipe (A4 paysage).
  - « Envoyer par Gmail » (planning global ou individuel), en un clic, sans serveur. Gmail ne permet pas à une page de joindre un fichier : l'envoi se fait donc en solution de repli. 1. Le PDF du planning affiché (A4 paysage, le calendrier du mois et les heures) est fabriqué par la page et **téléchargé aussitôt dans le dossier Téléchargements**, sans fenêtre. 2. **Gmail s'ouvre dans le navigateur**, dans un nouvel onglet, sur un message prérempli : destinataire, objet et texte. Destinataire : la personne pour un planning individuel, tous les salariés (sans le gérant) pour le planning global, d'après les adresses e-mail de l'onglet Équipe. 3. Un bandeau dit « **Le fichier a été téléchargé, glissez-le dans le mail** » ; il propose aussi « Rouvrir Gmail » et « Télécharger à nouveau ». Il reste à faire glisser le PDF (depuis les téléchargements de Chrome, en haut à droite) dans le message.
  - Sous le nom de chaque salarié, deux liens :
    - « **Fiche d'heures** » produit la fiche à faire signer (A4 portrait) : chaque jour avec ses créneaux, chacun précédé de son activité sur une étiquette de couleur (par exemple **Élevage** pour le matin et le soir, **Magasin** l'après-midi), et ses heures ; le total de chaque semaine ; les heures prévues, les heures à faire avec le détail des congés ; les heures de chaque activité ; le solde de congés ; deux encadrés de signature (employeur, employé) avec date.
    - « **Planning individuel** » ouvre le calendrier du planning enregistré filtré sur ce salarié, et sur son activité s'il n'en a qu'une. Deux listes changent le salarié (ou « Toute l'équipe ») et l'activité. « Imprimer ce planning » et « Envoyer par Gmail » portent sur la vue filtrée ; « Fermer » revient au planning complet.
- **Équipe** : nom, adresse e-mail (facultative, destinataire prérempli de l'envoi du planning), heures par mois, jours de repos par semaine, dimanches de repos par mois, solde de congés au 1er juin (de l'année de congés du mois affiché dans Planning), créneaux et jours possibles pour chacun. Cocher « Gérant » masque aussitôt les heures par mois, les repos et le solde de congés, qui ne le concernent pas ; décocher la case les fait réapparaître (151 h, 2 jours, 2 dimanches pour un ancien gérant). Un seul bouton « Enregistrer les modifications » pour toutes les fiches ; « Annuler les modifications » rend aux champs les valeurs enregistrées. Sur la fiche de chaque salarié, « **Historique des heures sup.** » affiche, mois par mois, l'écart du mois (▲ en plus, ▼ en moins), le cumul et l'état : mois validé (avec sa date), ou en cours (le mois qui suit le dernier mois validé : écart provisoire, recalculé à chaque changement du planning, hors cumul). Les mois planifiés plus loin sont ignorés. Le cumul des mois validés est rappelé en tête (`overtimeHistory()`).
- **Réglages** : titre (« Planning » par défaut), passage été / hiver, règles de répartition (à cocher et à classer par glisser-déposer), jours fériés, créneaux de travail (champs texte), sauvegarde. Un seul bouton « Enregistrer les réglages » ; « Annuler les modifications » abandonne tout ce qui n'est pas enregistré. La durée maximale d'une journée (10 h) n'est plus un réglage.
- **Encart de confirmation** : après un clic sur un bouton « Enregistrer » (Planning, Équipe, Réglages), un encart vert-bleu « ✓ … enregistré » apparaît environ 3 secondes en bas de l'écran, une fois l'écriture dans le fichier réussie. S'il n'y a rien à enregistrer, l'encart le dit.
- L'interrupteur **Administration**, en haut à droite, affiche ou masque les seuls onglets **Équipe** et **Réglages** ; les boutons du planning (générer, modifier, congés, valider le mois) restent toujours disponibles. Il est **décoché à chaque ouverture** de la page ; seule la toute première mise en place (création du fichier) l'ouvre coché.

Pièges connus :

- **Message « Ouvrez cette page avec Google Chrome »** : le navigateur utilisé ne sait pas écrire dans un dossier. Clic droit sur le fichier → « Ouvrir avec » → Google Chrome.
- **Écran « Autoriser l'accès » à chaque ouverture** : choisir « Autoriser à chaque visite » dans la fenêtre de Chrome.
- **La page, les données et le journal doivent être dans le même dossier.** La page prévient si ce n'est pas le cas.
- **« Enregistrement impossible »** : le disque ou le dossier est inaccessible. Les modifications restent à l'écran : rétablir l'accès, puis cliquer sur « Réessayer d'enregistrer ».
- **« Le fichier … est vide ou illisible »** : le fichier de données est abîmé. Rien n'a été modifié ; restaurer une sauvegarde.
- **Le message ne contient pas le PDF** : c'est normal, Gmail ne le permet pas depuis un lien. Faire glisser le fichier dans le message, ou le joindre avec le trombone.
- **Le PDF ne va pas dans Téléchargements** : dans les paramètres de Chrome (Téléchargements), l'option « Toujours demander où enregistrer les fichiers » est activée ; la désactiver.
- **L'onglet Gmail ne s'ouvre pas** : Chrome a bloqué la fenêtre ; cliquer sur « Rouvrir Gmail » dans le bandeau.
- **Impossible de modifier un mois** : il est validé (cadenas fermé). Cliquer sur « Mois validé » pour le déverrouiller.
- **Impression** : laisser les marges sur « Par défaut » dans la fenêtre d'impression de Chrome ; la page gère elle-même ses marges, et n'imprime ni date, ni titre, ni adresse de fichier.

### Enregistrement des modifications

Chaque modification enregistrée est appliquée en mémoire, puis le fichier de données est réécrit en entier ; les écritures passent l'une après l'autre. Le message « Enregistré à hh:mm:ss » confirme l'écriture.

Le journal `planning-elevage.log` reçoit une ligne par modification, seulement une fois l'écriture réussie. Une modification sans effet n'ajoute rien au journal, et « Enregistrer les modifications » sur des fiches inchangées n'écrit rien du tout.

---

## 6. Fichiers, structure et modification de l'outil

### Structure du projet

```
planning-elevage.html          La page : interface et logique
  feuille de style             Écran et impression (A4 paysage pour le calendrier, portrait pour la fiche)
  Core                         Logique métier pure : dates, saisons, créneaux, données, journal,
                               enregistrement, répartition automatique, analyse et alertes
  Interface                    État de l'écran, rendu, clics, accès au dossier, impression
README.md                      Ce document
.gitignore                     Exclut les fichiers de données et les sauvegardes du dépôt
```

`Core` ne touche jamais à l'écran et l'interface ne contient aucune règle métier : une règle de calcul se modifie dans `Core`, et se vérifie sans navigateur.

Le code est écrit en anglais : variables, fonctions, classes CSS, actions des boutons et **clés du fichier de données**. Les commentaires, les textes affichés et le journal sont en français. Vocabulaire : `shift` = créneau, `manager` = gérant, `employees` = équipe, `schedules` = plannings, `leaveDays` = congés.

### Fichiers de données

La page crée deux fichiers dans son dossier. Ils contiennent des données personnelles et ne doivent pas être versés dans le dépôt (voir `.gitignore`).

```
planning-elevage-donnees.json  Réglages, créneaux, équipe, congés, plannings de chaque mois, mois validés
planning-elevage.log           Journal des modifications, une ligne par action
```

Exemple de contenu (valeurs fictives), version 2 du format :

```json
{
 "format": "planning-elevage",
 "version": 2,
 "settings": { "title": "Planning", "seasonMode": "clock",
               "summerStart": "04-01", "winterStart": "10-01",
               "rules": [ { "id": "sundaysOff", "on": true }, { "id": "restPair", "on": true },
                          { "id": "sundaySpread", "on": true } ],
               "holidays": [ { "date": "2026-12-25", "name": "Noël" } ] },
 "shifts": [ { "id": "p…", "activity": "Élevage", "days": [1,2,3,4,5,6,7],
               "winter": { "from": "09:00", "to": "13:00" }, "summer": { "from": "08:00", "to": "12:00" },
               "min": 1, "max": 3, "managerNeverAlone": true, "showEmpty": true,
               "neverEmpty": { "summer": false, "winter": false } } ],
 "employees": [ { "id": "s…", "name": "Salarié A", "email": "salarie.a@exemple.fr", "monthlyHours": 151, "manager": false,
                  "restDaysPerWeek": 2, "sundaysOffPerMonth": 2, "shifts": ["p…"],
                  "shiftDays": { "p…": [1,2,4,5,6] }, "leaveDays": ["2026-12-14", "2026-12-15"], "blockedSlots": ["2026-12-09|p…"],
                  "leaveStart": { "2026": 25 } } ],
 "removedIds": [],
 "schedules": { "2027-01": { "assignments": { "2027-01-04|p…": ["s…"] }, "locks": [], "savedAt": "…",
                             "validated": { "at": "…", "balance": { "s…": -3 } } } },
 "log": [],
 "updatedAt": "…"
}
```

| Clé | Contenu |
|---|---|
| `settings` | Réglages : titre (l'ancienne clé `maxDayHours` n'est plus lue : 10 h, constante), passage été / hiver (`seasonMode` : `clock` au changement d'heure, `dates` à dates fixes `summerStart` / `winterStart`, au format MM-JJ), règles de répartition (`rules` : `{id, on}` dans l'ordre de priorité ; une règle absente est ajoutée à la fin, cochée), jours fériés (`holidays` : `{date, name}` ; un congé ces jours-là n'est pas décompté) |
| `shifts` | Créneaux (sans nom) : activité, jours (1 = lundi … 7 = dimanche), horaires d'hiver et d'été (`from`, `to` ; `null` = pas de créneau cette saison), personnes au moins / au plus, gérant jamais seul, `showEmpty` (affiché même vide), `neverEmpty` (`{summer, winter}` : à ne jamais laisser vide). Une clé `managerDays` d'une version précédente est reprise sur la fiche du gérant, puis n'est plus écrite |
| `employees` | Personnes : nom, `email` (facultatif), heures du contrat par mois, gérant ou non, jours de repos par semaine, dimanches de repos par mois, créneaux autorisés (pour le gérant : ses jours de présence fixes), `shiftDays` (jours où la personne peut tenir un créneau ; un créneau absent de cette liste est possible tous ses jours), `leaveDays` (jours de congé), `blockedSlots` (créneaux indisponibles hors congés, `"AAAA-MM-JJ|créneau"`, non décomptés ; l'ancienne clé `blockedDays` est reprise en tous les créneaux du jour), `leaveStart` (solde de congés au 1er juin, en jours, par année : `"2026"` = juin 2026 → mai 2027) |
| `removedIds` | Identifiants des personnes retirées : jamais réutilisés |
| `schedules` | Plannings par mois : `assignments` (« date\|créneau » → personnes), `locks` (jours gardés à la main), `savedAt`, `validated` (`null`, ou mois validé : date et écart de chaque salarié, en heures) |
| `log` | Toujours vide dans le fichier : tampon avant l'écriture du journal `.log` |

**Créneaux d'une version antérieure** : un créneau avait un nom (« Matin »). Il n'en a plus : à la lecture, un créneau sans activité prend son ancien nom comme activité, et le nom n'est plus réécrit dans le fichier.

**Fichier d'une version antérieure** : un fichier de la version 1, aux clés en français (`reglages`, `plages`, `salaries`, `plannings`…), s'ouvre normalement. `fromLegacy()` renomme ses clés à la lecture, et il est réécrit au format 2 à la prochaine modification. Les activités `elevage` et `boutique` deviennent « Élevage » et « Magasin » ; les identifiants des créneaux et des personnes ne changent pas, ni les plannings.

Une ligne du journal (valeurs fictives) :

```
2027-01-04 09:12:45 | Administration | Planning de janvier 2027 enregistré : Salarié A 148 h, Salarié B 84 h, Gérant présent 31 j [#smwy5a4386zrloy]
```

Pour restaurer une sauvegarde, la replacer dans le dossier sous le nom `planning-elevage-donnees.json`.

### Modifier l'outil

| Je veux… | Où intervenir |
|---|---|
| Changer le passage été / hiver | `isSummer()` |
| Reprendre un ancien format de fichier | `fromLegacy()`, appelée au début de `normalize()` |
| Changer la valeur d'un jour de congé | `leaveDayHours()`, et le plafond par semaine dans `weekRules()` |
| Changer le calcul des heures à faire | `targetOf()`, utilisée par `analyze()` (affichage) et `propose()` (génération) |
| Changer le compteur d'heures sup. | `carryBefore()`, l'écart noté par `validateMonth` dans `Core.applyOp()`, et `overtimeOf()` pour la carte |
| Changer l'année ou le décompte des congés | `leavePeriod()`, `leaveBalance()` |
| Modifier les priorités de la répartition | `cost()` dans `Core.propose()` ; ajouter une règle de répartition : `RULES`, son calcul dans `cost()`, son alerte dans `analyze()` |
| Changer le message Gmail | `startMail()`, `mailRecipients()` et `gmailUrl()` |
| Changer le PDF envoyé par Gmail | `planningPdf()` (mise en page), `pdfDoc()` (écriture du fichier) |
| Ajouter une alerte | `Core.analyze()`, puis `checksCard()` pour l'affichage |
| Ajouter un réglage | `defaultSettings()`, `normalize()`, `SETTING_NAMES`, `settingsView()`, action `save-all-settings` |
| Ajouter un type de modification | un `case` dans `Core.applyOp()`, qui doit décrire un état voulu et rester sans effet s'il est rejoué |

## Contributeurs

### Développeur

Réalisation et mise à disposition de l'outil à partir du besoin exprimé par l'exploitation :

- traduction du besoin en règles et en fonctionnalités ;
- direction du travail de l'agent de code, relecture et validation ;
- publication du projet.

### Agent de code

Claude Opus 5.5, via Claude Code, sous la direction du développeur : écriture de `planning-elevage.html` à partir de *Permanences du week-end* (interface, règles, congés, répartition automatique, alertes, impression, enregistrement), commentaires d'audit du code et rédaction de ce README.

## Licence

Ce projet dérive de *Permanences du week-end* (`duty-roster-planner`), publié sous [GNU General Public License, version 3](https://www.gnu.org/licenses/gpl-3.0.html). Il est distribué sous la même licence.
