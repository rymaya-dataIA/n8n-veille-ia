# Guide d'Installation
## Workflow N8N — Veille IA Automatique

---

## Table des matieres

1. [Vue d'ensemble](#1-vue-densemble)
2. [Prerequis](#2-prerequis)
3. [Importer le workflow](#3-importer-le-workflow)
4. [Configurer OpenRouter](#4-configurer-openrouter)
5. [Configurer Gmail](#5-configurer-gmail)
6. [Tester le workflow](#6-tester-le-workflow)
7. [Activer l'automatisation](#7-activer-lautomatisation)
8. [Resolution des problemes](#8-resolution-des-problemes)
9. [Personnalisation avancee](#9-personnalisation-avancee)

---

## 1. Vue d'ensemble

Ce workflow surveille 5 sources RSS specialisees en intelligence artificielle, filtre les articles des dernieres 24 heures, genere un resume structure en francais via un modele IA gratuit, puis envoie le tout dans un seul email chaque matin a 6h00.

**Duree d'installation estimee : 20 minutes**
**Cout total : 0 euro**

---

## 2. Prerequis

Avant de commencer, verifie que tu disposes de :

- Un compte N8N (cloud sur [n8n.io](https://n8n.io) ou self-hosted)
- Un compte Google personnel pour Gmail
- Un compte OpenRouter gratuit sur [openrouter.ai](https://openrouter.ai)

---

## 3. Importer le workflow

**Etape 1** — Connecte-toi a ton instance N8N.

**Etape 2** — Clique sur **New Workflow** dans le menu principal.

**Etape 3** — Ouvre le menu en haut a droite (3 points) et selectionne **Import from file**.

**Etape 4** — Selectionne le fichier `workflow_veille_ia_v8.json`.

Le workflow s'ouvre avec les 14 noeuds deja connectes et configures.

---

## 4. Configurer OpenRouter

OpenRouter est une plateforme qui donne acces a des modeles IA gratuits via une API standard.

### Creer une cle API

**Etape 1** — Va sur [openrouter.ai](https://openrouter.ai) et cree un compte gratuit.

**Etape 2** — Dans le menu, clique sur **Keys** puis **Create Key**.

**Etape 3** — Donne un nom a ta cle, par exemple `n8n-veille-ia`.

**Etape 4** — Copie la cle affichee. Elle est au format `sk-or-v1-...`

> Important : sauvegarde cette cle dans un endroit sur. Elle ne sera plus visible apres la fermeture de la page.

### Configurer le noeud dans N8N

**Etape 1** — Dans le workflow, clique sur le noeud **OpenRouter - Resume IA**.

**Etape 2** — Va dans l'onglet **Parameters**.

**Etape 3** — Dans la section **Headers**, trouve la ligne **Authorization**.

**Etape 4** — Dans le champ **Value**, remplace `VOTRE_CLE_API_OPENROUTER` par :

```
Bearer sk-or-v1-TACLÉ
```

Le mot `Bearer` suivi d'un espace avant la cle est obligatoire.

> Rappel : ne publie jamais ta cle API dans un depot GitHub public.

---

## 5. Configurer Gmail

### Creer la connexion Gmail dans N8N

**Etape 1** — Dans N8N, va dans **Settings** puis **Credentials**.

**Etape 2** — Clique sur **Add Credential** et cherche **Gmail OAuth2 API**.

**Etape 3** — Clique sur **Sign in with Google** et autorise N8N a acceder a ton compte.

### Configurer le noeud Email

**Etape 1** — Dans le workflow, clique sur le noeud **Envoyer Email Digest**.

**Etape 2** — Dans le champ **Credential**, selectionne le compte Gmail que tu viens de connecter.

**Etape 3** — Dans le champ **To**, remplace `VOTRE_EMAIL@exemple.com` par ton adresse email.

---

## 6. Tester le workflow

Avant d'activer l'automatisation, effectue un test complet.

**Etape 1** — Clique sur **Execute Workflow** en haut de l'ecran.

**Etape 2** — Attends que tous les noeuds affichent un indicateur de succes.

**Etape 3** — Verifie ta boite email dans les 2 minutes suivantes.

### Resultats attendus par noeud

| Noeud | Resultat attendu |
|-------|-----------------|
| RSS (5 sources) | Articles avec titre, lien et date |
| Merge Final | Ensemble des articles combines |
| Filtre Date 24h | Articles des dernieres 24h uniquement |
| Limit 15 articles | 15 articles au maximum |
| Aggregate | 1 item contenant la liste des articles |
| Prepare Texte | 1 item avec les champs `texte` et `count` |
| OpenRouter | Reponse JSON avec le resume genere |
| Extrait Resume | 1 item avec le champ `resume` formate |
| Envoyer Email Digest | Email envoye avec succes |

> Note : si le Filtre Date retourne 0 articles (souvent le lundi apres un week-end), c'est normal car peu d'articles sont publies le dimanche. Pour tester, modifie temporairement le filtre a 3 jours en remplacant `minus(1, 'days')` par `minus(3, 'days')`.

---

## 7. Activer l'automatisation

Une fois le test reussi :

**Etape 1** — Clique sur le toggle **Inactive** en haut a droite du workflow.

**Etape 2** — Le statut passe a **Active**.

Le workflow se declenchera desormais automatiquement chaque matin a 6h00.

---

## 8. Resolution des problemes

### Erreur 404 sur un noeud RSS

**Cause** : L'URL du flux RSS a change ou n'est plus disponible.

**Solution** : Remplace l'URL par une alternative. Consulte la section 9 pour des exemples.

### Erreur 401 sur OpenRouter — Missing Authentication

**Cause** : La cle API est absente ou mal formatee.

**Solution** : Verifie que la valeur du header Authorization est exactement :
```
Bearer sk-or-v1-TACLÉ
```
Le mot `Bearer` et l'espace avant la cle sont obligatoires.

### Erreur 429 sur Gmail — Rate limit exceeded

**Cause** : Trop de tests effectues en peu de temps. Gmail limite les envois rapproches.

**Solution** : Attends 20 a 30 minutes avant de relancer. En usage normal avec un seul email par jour, ce probleme ne se produit pas.

### L'email recu est vide — pas de resume visible

**Cause** : Le noeud Extrait Resume ne trouve pas le bon champ dans la reponse de l'IA.

**Solution** : Clique sur le noeud OpenRouter et verifie que l'OUTPUT contient bien le champ `choices[0].message.content` avec du texte a l'interieur.

### Le resume est en anglais ou tronque

**Cause** : Le modele IA n'a pas suivi les instructions ou manque de tokens pour finir.

**Solution** : Dans le noeud OpenRouter, augmente la valeur de `max_tokens` a 2000.

---

## 9. Personnalisation avancee

### Sources RSS alternatives

```
MIT Technology Review AI  : https://www.technologyreview.com/topic/artificial-intelligence/feed
TechCrunch AI             : https://techcrunch.com/category/artificial-intelligence/feed
Arxiv Machine Learning    : https://rss.arxiv.org/rss/cs.LG
The Verge AI              : https://www.theverge.com/rss/ai-artificial-intelligence/index.xml
```

### Modifier la frequence d'envoi

Dans le noeud **Schedule Trigger**, modifie l'expression cron :

```
0 6 * * *    — tous les jours a 6h00 (configuration par defaut)
0 7 * * *    — tous les jours a 7h00
0 8 * * 1-5  — du lundi au vendredi a 8h00
```

### Modifier la periode du filtre de date

Dans le noeud **Filtre Date 24h**, modifie la valeur dans la condition :

```javascript
// 24 heures — configuration par defaut
$now.minus(1, 'days').toMillis()

// 3 jours — recommande si peu d'articles disponibles
$now.minus(3, 'days').toMillis()
```

### Changer le modele IA

Dans le noeud **OpenRouter - Resume IA**, remplace la valeur du champ `model` dans le body :

```
z-ai/glm-4.5-air:free              — modele par defaut
google/gemma-3-12b-it:free         — alternative Google
mistralai/mistral-7b-instruct:free — alternative Mistral
```

La liste complete des modeles gratuits est disponible sur [openrouter.ai/models](https://openrouter.ai/models?q=free).

---

*Documentation creee dans le cadre du programme Ambassadrice N8N.*
