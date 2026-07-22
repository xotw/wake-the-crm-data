# Wake the CRM — Challenge GTM Engineer

Bienvenue. Ce repo contient tout ce qu'il te faut. Lis ce README en entier avant d'ouvrir les CSVs.

---

## Le scenario

Tu rejoins une scale-up HR-tech (fictive) qui vend un logiciel RH (ATS, performance, paie) en Europe. Son CRM a 8 ans d'historique et personne ne l'a jamais entretenu :

- **30 000 comptes** et **77 000 contacts**, accumules par 3 generations de sales.
- Dedans, melanges : des clients actifs, des clients churnes, des prospects morts, des doublons, des champs contradictoires.
- Et surtout : **quelque part dans cette masse, des comptes montrent des signaux d'achat en ce moment meme — et personne ne les regarde.**

Ta mission : **construire la machine qui les trouve.**

## Ce que tu recois

| Fichier | Contenu |
|---|---|
| `accounts.csv` | 30 000 comptes — le CRM brut, avec tous ses defauts |
| `contacts.csv` | 77 000 contacts rattaches (ou pas) aux comptes |
| `events.csv` | ~95 000 evenements d'engagement sur les 90 derniers jours |

Le dictionnaire des donnees est en bas de ce README.

**Important :** ces donnees sont 100 % synthetiques, mais leurs defauts sont realistes et **volontaires**. Rien n'est propre. C'est le jeu.

---

## Partie 1 — La machine (le gros du travail)

### 1. Cleanup d'abord

Un scoring sur des donnees sales ne vaut rien. Avant de scorer :

- Dedoublonne les comptes (il y a des doublons — noms differents, domaines differents, memes entreprises).
- Segmente la base : client actif / churne / prospect / mort.
- Resous les contradictions que tu trouveras (il y en a).

### 2. Le scoring engine — le coeur du test

Rien n'est impose. C'est TOI qui decides, et tu devras defendre chaque choix :

- **Ta taxonomie de triggers.** Qu'est-ce qui compte comme un signal ? Comment distingues-tu le fit (ce compte vaut-il quelque chose) de l'engagement (bougent-ils maintenant) ? Un event isole vs un cluster ? Est-ce que 3 contacts actifs en 7 jours = 1 contact actif 3 fois ? Un click d'il y a 60 jours vaut-il encore quelque chose ? Et les signaux negatifs ?
- **Ou vit ton scoring.** Dans un outil ? Dans une base externe ? Pourquoi ?
- **Config, pas code.** Les poids et seuils doivent etre modifiables **sans redeployer**. On te demandera de changer une regle en live.

### 3. Le push

Les comptes chauds doivent atterrir quelque part d'actionnable (digest Slack, webhook, ce que tu veux) — **avec les preuves** : « Compte X passe hot : 3 contacts actifs cette semaine dont la VP People, /pricing visite 2 fois, demo request. »

### 4. Le dashboard — deploye sur ton infra

Trois vues :

- **Vue macro.** Le CRM comme une masse endormie qui se reveille : les comptes s'allument au fur et a mesure que les events arrivent. Les events sont timestampes — tu peux rejouer le stream en live pendant la demo. Sois creatif, on veut du spectaculaire ET du lisible.
- **Vue logique.** Clic sur un compte → tous ses signaux, chaque regle declenchee, la composition du score, le decay applique. La question « pourquoi ce compte est-il hot ? » doit trouver sa reponse **dans l'UI seule**, sans ouvrir le code.
- **Vue ops.** Minimale : statut du dernier run + une alarme si quelque chose casse.

### Ce qu'on evalue (partie 1)

On connait les reponses : les comptes reellement chauds ont ete plantes dans les donnees, ainsi que **des pieges** concus pour tromper un scoring naif. Ta note, c'est :

1. **Precision et rappel** — combien de vrais chauds trouves, combien de faux positifs remontes.
2. **Les pieges** — y es-tu tombe ?
3. **Explicabilite** — chaque score justifiable depuis le dashboard.
4. **Le test live** — changer un poids sans redeployer, ingerer un event pourri sans crash.

---

## Partie 2 — L'open challenge (demo day)

> **« Montre-nous l'outil GTM qui manque aujourd'hui — soit un truc que tu as deja construit, soit ce que tu construirais ensuite. Deroule la logique entiere : quel probleme, pourquoi maintenant, comment ca marche bout en bout, ou ca casse. »**

Pas besoin d'un build poli. Le livrable, c'est ton raisonnement + un schema + (optionnel) un prototype rugueux.

Evalue sur 4 axes :

| Axe | La question qu'on se posera |
|---|---|
| **Pragmatisme** | Est-ce que ca shippe en 2 semaines, et quelqu'un l'utilise lundi ? |
| **Enthousiasme** | Est-ce que tu y crois vraiment, ou c'est une idee tendance LinkedIn ? |
| **Avant-garde** | Y a-t-il UN move genuinement non evident dedans ? |
| **Excellence** | Ta logique survit-elle a nos questions ? Sais-tu ou ca casse ? |

---

## Regles du jeu

- **Duree : 1 semaine**, demo a la fin. Rythme indicatif : J1-J2 cleanup + modele de donnees, J3-J4 scoring + push, J5 dashboard.
- **Stack libre.** On juge le resultat et la logique, pas les outils.
- **Deploye**, pas en localhost. Ton infra, ton choix.
- **IA bienvenue.** Utilise Claude, GPT, ce que tu veux — mais tu dois pouvoir defendre chaque ligne de logique comme si tu l'avais ecrite.
- Des questions ? Pose-les. Un GTM engineer qui n'interroge pas le brief est plus inquietant qu'un qui le challenge.

---

## Dictionnaire des donnees

### accounts.csv

| Champ | Note |
|---|---|
| `account_id` | Identifiant unique du record (pas de l'entreprise…) |
| `account_name` | Nom — formats incoherents |
| `domain` | Parfois vide, parfois prefixe www. |
| `industry`, `employee_range` | Parfois vides |
| `country` | Formats incoherents (FR / France / france…) |
| `lifecycle_stage` | lead / prospect / opportunity / customer / churned / lost — pas toujours fiable |
| `created_date`, `last_activity_date` | **Trois formats de date coexistent** |
| `owner` | Sales owner, parfois vide |
| `arr_eur` | Rempli pour customers (et churned) |
| `renewal_date` | Rempli pour customers… normalement |

### contacts.csv

| Champ | Note |
|---|---|
| `contact_id`, `account_id` | Des contacts orphelins existent (account_id vide) |
| `email` | Parfois vide, malforme, ou perso (gmail) |
| `job_title` | Du CHRO au stagiaire — a toi de definir qui compte |
| `opted_out` | true/false — reflechis a ce que ca implique |
| `created_date` | Memes formats incoherents |

### events.csv

| Champ | Note |
|---|---|
| `event_id`, `timestamp` | ISO 8601, tries chronologiquement, fenetre de 90 jours |
| `event_type` | email_sent / email_open / email_click / form_fill / meeting_booked / website_visit / linkedin_engagement |
| `contact_id` | Vide pour les visites anonymes |
| `account_id` | Present quand connu — attention, c'est l'id du **record**, pas de l'entreprise |
| `campaign` | Pour les events email |
| `page_url` | Pour les visites web — toutes les pages ne se valent pas |
| `metadata` | Contexte libre |

Bonne chasse.
