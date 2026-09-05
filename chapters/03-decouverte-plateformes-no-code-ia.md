## Panorama des plateformes No‑Code IA

Le marché du no‑code propose aujourd’hui des environnements visuels capables d’appeler directement des modèles d’intelligence artificielle. Six outils se distinguent par leur popularité en Afrique francophone et par la richesse de leurs extensions IA :

| Plateforme | Type d’application | Points forts pour l’Afrique |
|------------|-------------------|-----------------------------|
| **Bubble** | Web app (desktop & mobile) | Hébergement multi‑région, API Connector très complet, communauté active francophone. |
| **Adalo** | Apps mobiles natives (iOS/Android) | Publication directe sur les stores, support offline limité mais suffisant pour des apps simples. |
| **Softr** | Portails, marketplaces, intranets (sur Airtable ou Google Sheets) | Déploiement ultra‑rapide, plans gratuits généreux, intégration native de Zapier/Make. |
| **Parabola** | ETL visuel, transformation de données | Idéal pour nettoyer des jeux de données provenant de réseaux ruraux (CSV, Excel, API). |
| **Zapier** | Automatisation d’événements (« if this then that ») | Plus de 3 000 applications connectées, large bibliothèque de « Zaps » IA. |
| **Make (Integromat)** | Scénarios d’automatisation avancés, branchements conditionnels | Possibilité de créer des boucles et des agrégations, tarif à la minute d’exécution, très efficace côté coût. |

Toutes ces plateformes offrent un **module d’appel d’API** (ou « Webhooks ») qui permet d’interroger les services d’OpenAI, Cohere ou Hugging Face. La différence réside surtout dans la façon dont on construit l’interface utilisateur : Bubble et Adalo offrent un éditeur visuel complet, Softr se base sur des tables de données, tandis que Zapier/Make orchestrent les flux entre plusieurs services.

---

## Créer un compte et préparer l’environnement

### Inscription et plan adapté

1. **Bubble** – Rendez‑vous sur `bubble.io`, cliquez sur *Sign up*. L’adresse e‑mail peut être une adresse Gmail ou une adresse locale (ex. : `nom@orange.fr`). Le plan *Free* permet jusqu’à 2 000 actions API/mois, largement suffisant pour les premiers tests.  
2. **Adalo** – `adalo.com` propose un plan gratuit limité à 50 utilisateurs actifs. La facturation se fait par carte bancaire ; les cartes locales (Visa/Mastercard) émises par les banques africaines sont acceptées, mais il est parfois plus simple d’utiliser **PayPal** ou **Flutterwave** (option « Pay with mobile money » disponible).  
3. **Softr** – `softr.io` accepte les paiements par **M-Pesa** (Kenya, Tanzanie) et **Orange Money** (Côte d’Ivoire, Sénégal) dès le plan *Pro*. Le plan *Free* donne accès à 100 enregistrements et à 1 000 exécutions d’API/mois.  
4. **Parabola** – `parabola.io` propose un essai de 14 jours sans carte. Le plan *Starter* (10 000 lignes de données/mois) convient aux PME agricoles qui manipulent des fichiers CSV de petite taille.  
5. **Zapier** – `zapier.com` accepte les cartes internationales ; pour les entrepreneurs sans carte, le **code promo** « AFRICA30 » permet de créer un compte gratuit avec 100 tasks/mois.  
6. **Make** – `make.com` propose un plan gratuit de 1 000 operations/mois. Le paiement se fait via **Stripe** ou **PayPal** ; les entrepreneurs peuvent contourner l’absence de carte en passant par un compte Stripe lié à un compte bancaire local (ex. : **Paystack** en Nigéria).

### Configuration du tableau de bord

- **Définir un espace de travail** : chaque plateforme utilise le terme *project*, *app* ou *scenario*. Créez un projet nommé `IA‑Africain‑Demo`.  
- **Activer les variables d’environnement** : stockez les clés API dans les paramètres sécurisés (ex. : *Bubble → Settings → API → Private keys*). Cela évite de les exposer dans le code visuel.  
- **Choisir la zone géographique** : Bubble et Make permettent de sélectionner un data‑center (Europe ou US). Pour minimiser la latence en Afrique, choisissez le **Europe‑West** (Ireland) qui est généralement le plus proche.

---

## Connecter les API d’IA

### Gestion des clés d’API

| Service | Point d’obtention | Exemple de clé (masquée) |
|---------|-------------------|--------------------------|
| **OpenAI** | `platform.openai.com/account/api-keys` | `sk-********************` |
| **Cohere** | `dashboard.cohere.com/api-keys` | `cohere-****************` |
| **Hugging Face** | `huggingface.co/settings/tokens` | `hf_*****************` |

Conservez chaque clé dans la zone **secrète** de la plateforme (ex. : *Bubble → Settings → Secrets*).  

### Exemple de connexion dans Bubble (API Connector)

1. Ouvrez le plugin **API Connector** et cliquez sur *Add another API*.  
2. Nommez l’API : `OpenAI‑Chat`.  
3. **Authentication** : choisissez *Private key in header* et indiquez `Authorization: Bearer <your_openai_key>`.  
4. **Endpoint** : `https://api.openai.com/v1/chat/completions`.  
5. Méthode : `POST`.  
6. Corps de la requête (JSON) :

```json
{
  "model": "gpt-4o-mini",
  "messages": [
    {"role": "system", "content": "Tu es un assistant qui rédige des descriptions de produits en français."},
    {"role": "user", "content": "<title>"}
  ],
  "max_tokens": 150,
  "temperature": 0.7
}
```

7. Cochez *Use as Action* pour pouvoir appeler l’API depuis un bouton ou un workflow.

### Exemple de connexion dans Zapier (Webhooks)

1. Créez un *Zap* → **Trigger** : *New Row in Google Sheets* (liste de produits).  
2. Action : **Webhooks by Zapier** → *Custom Request*.  
3. Paramètres :

| Champ | Valeur |
|------|--------|
| Method | POST |
| URL | `https://api.cohere.com/v1/generate` |
| Data | `{"model":"command","prompt":"Rédige une description courte pour le produit suivant : {{Titre}}","max_tokens":100}` |
| Headers | `Authorization: Bearer {{Cohere_API_Key}}`<br>`Content-Type: application/json` |

4. Ajoutez une seconde action *Update Row* pour enregistrer la réponse dans la même feuille.

### Exemple de connexion dans Make (module HTTP)

1. Dans un scénario, ajoutez le module **HTTP > Make a request**.  
2. Configuration :  

| Paramètre | Valeur |
|-----------|--------|
| URL | `https://api-inference.huggingface.co/models/facebook/bart-large-cnn` |
| Method | POST |
| Headers | `Authorization: Bearer {{HF_TOKEN}}`<br>`Content-Type: application/json` |
| Body | `{"inputs":"{{texte}}","parameters":{"max_length":200}}` |

3. Connectez le module à une source de données (ex. : *Google Sheets – Get a row*).  
4. Le résultat (`{{response.body}}`) peut être envoyé à **WhatsApp Business API** ou stocké dans **Airtable**.

### Utilisation de modèles pré‑entraînés

- **ChatGPT** : génération de texte libre, réponses conversationnelles.  
- **Cohere command** : texte court, idéal pour des résumés ou des titres.  
- **Hugging Face summarization** : réduction de rapports d’études de marché (utile pour les ONG).  

Choisissez le modèle en fonction du **coût par token** et de la **qualité attendue**. En Afrique, où les budgets sont souvent serrés, le modèle `gpt-4o-mini` (ou `cohere command`) représente le meilleur compromis.

---

## Construire son premier workflow IA

### Cas d’usage : génération automatique de description produit

Un commerçant de **kitenge** (tissu africain) veut enrichir son catalogue en ligne. Chaque fois qu’il saisit le nom du produit, l’IA crée une description de 150 mots prête à être affichée.

#### Étape 1 – Capture du titre (Bubble)

- Créez une page `Produit` avec un *Input* nommé `inputTitre`.  
- Ajoutez un bouton **Générer description**.

#### Étape 2 – Appel à l’API OpenAI

Dans le workflow du bouton :

1. Action : *Plugins → OpenAI‑Chat – Call*.  
2. Paramètre `title` : `inputTitre's value`.  
3. Stockez le résultat dans une variable temporaire `descriptionIA`.

#### Étape 3 – Sauvegarde dans la base de données

- Créez un type de données `Produit` avec les champs `titre` (texte) et `description` (texte).  
- Action : *Data → Create a new thing* → `Produit` : `titre = inputTitre's value`, `description = descriptionIA`.

#### Étape 4 – Notification via WhatsApp

- Ajoutez une action *API Connector* : appel à **Twilio** (`https://api.twilio.com/2010-04-01/Accounts/{{AccountSid}}/Messages.json`).  
- Corps : `Body=Votre produit "{{inputTitre's value}}" a été enrichi.`, `To=+{{num_telephone}}`, `From=+{{twilio_number}}`.  

Le commerçant reçoit immédiatement un SMS/WhatsApp confirmant la création.

### Implémentation dans Adalo (action custom)

1. Dans le **Screen** du formulaire produit, ajoutez un **Custom Action**.  
2. Configurez l’URL : `https://api.openai.com/v1/chat/completions`.  
3. En *Headers* : `Authorization: Bearer {{OPENAI_KEY}}`.  
4. En *Body* : même JSON que précédemment, avec la variable `{{title}}`.  
5. Mappez la réponse (`choices[0].message.content`) dans le champ `description` de la collection `Products`.

### Implémentation dans Softr (via Zapier)

1. **Trigger** : *New Record in Airtable* (table `Produits`).  
2. **Action** : *Webhooks – POST* à OpenAI (voir configuration Bubble).  
3. **Action** : *Update Record* : remplissez le champ `description` avec le texte retourné.  
4. Le site Softr affiche automatiquement la nouvelle description grâce à la liaison directe avec Airtable.

---

## Critères de sélection adaptés aux contraintes africaines

| Critère | Pourquoi c’est crucial | Astuce pratique |
|---------|------------------------|-----------------|
| **Connectivité** | Les réseaux 3G/4G sont parfois instables ; chaque appel API doit être rapide et résilient. | Préférez les plateformes qui offrent **caching** (ex. : Make – *Cache* module) et la possibilité de **retry** automatique. |
| **Coût** | Les modèles d’IA sont facturés à la tokenisation ; les micro‑entreprises ne peuvent pas absorber des dépenses imprévues. | Utilisez les **plans gratuits** pour tester, puis passez à un **plan à la consommation** (ex. : Zapier – *Pay‑as‑you‑go*). Limitez les appels à 1 fois par produit. |
| **Support local / communauté francophone** | Une aide en français accélère le déploiement. | Rejoignez les groupes Facebook « Bubble Africa Francophone », les forums **Discord** de **Make Africa** et les **Slack** de **Adalo Africa**. |
| **Hébergement et souveraineté des données** | Certaines industries (finance, santé) exigent que les données restent sur le continent. | Combinez **Bubble** (hébergement EU) avec **Cloudinary Africa** pour les médias, ou utilisez **Mali Cloud** (ex. : **Scaleway Africa**) comme point de terminaison d’API via un proxy. |
| **Scal