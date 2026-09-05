## Choisir le bon fournisseur cloud africain  

Le continent possède aujourd’hui trois grands fournisseurs de cloud public avec des zones de présence en Afrique : **AWS (Cape Town & Johannesburg), Microsoft Azure (South Africa North & West) et Google Cloud (South Africa West & East)**.  
Le choix dépend de trois critères :

| Critère | AWS | Azure | Google Cloud |
|--------|-----|-------|--------------|
| **Couverture** | 2 zones (ZA) – bonne latence pour l’Afrique du Sud, le Nigeria, le Kenya via le réseau *Edge* | 2 zones (ZA) + 1 zone (Nigeria) – intégration native avec Office 365, très apprécié des entreprises publiques | 2 zones (ZA) – IA intégrée (Vertex AI) et tarification attractive sur le stockage |
| **Tarification** | Facturation à la seconde, crédits gratuits 12 mois, **Free Tier** généreux (Lambda, S3) | Facturation à la minute, **Azure for Start‑ups** offre 120 000 $ de crédit | Facturation à la seconde, **Always Free** (Cloud Functions, Firestore) |
| **Écosystème No‑Code** | Amplify Console, API Gateway + Lambda, **Amazon Bedrock** (modèles IA) | Azure Logic Apps, Power Automate, **Azure OpenAI Service** | Cloud Run, Firebase Hosting, **Vertex AI** (API IA) |

Pour un entrepreneur qui démarre, le **Free Tier** d’AWS ou le programme **Azure for Start‑ups** sont souvent les plus simples à activer. Le facteur décisif reste la proximité géographique : un serveur hébergé à Cape Town servira mieux les utilisateurs du sud de l’Afrique que celui situé aux États‑Unis.

---

## Déployer une application No‑Code sur AWS Africa  

### 1. Préparer l’environnement  

1. Créez un compte AWS et choisissez la région **africa‑south‑1 (Cape Town)**.  
2. Activez **AWS Amplify** (service d’hébergement front‑end) et **API Gateway** (exposition d’APIs REST/GraphQL).  
3. Si votre solution utilise un modèle IA (ex. OpenAI), créez une **clé d’accès** dans **IAM** avec les permissions `AmazonBedrockReadOnly` ou `AmazonSageMakerFullAccess` selon le service choisi.

### 2. Publier le front‑end avec Amplify  

```yaml
# amplify.yml – fichier de build utilisé par Amplify
version: 1
frontend:
  phases:
    preBuild:
      commands:
        - npm ci
    build:
      commands:
        - npm run build
  artifacts:
    baseDirectory: /dist
    files:
      - '**/*'
  cache:
    paths:
      - node_modules/**/*
```

1. Connectez votre dépôt GitHub/GitLab à Amplify.  
2. Amplify crée automatiquement un **domain‑specific URL** (`https://master.d1a2b3c4.amplifyapp.com`).  
3. Ajoutez un **custom domain** (ex. `app.mabusiness.africa`) via Route 53 ou votre registrar local.

### 3. Exposer les fonctions IA avec API Gateway + Lambda  

```python
# lambda_function.py – fonction simple qui interroge OpenAI
import json, os, openai

openai.api_key = os.getenv('OPENAI_API_KEY')

def lambda_handler(event, context):
    prompt = json.loads(event['body'])['prompt']
    response = openai.Completion.create(
        model="gpt-3.5-turbo",
        prompt=prompt,
        max_tokens=150
    )
    return {
        'statusCode': 200,
        'body': json.dumps({'answer': response.choices[0].text.strip()})
    }
```

1. Créez la fonction Lambda (runtime Python 3.10) dans la même région.  
2. Ajoutez la variable d’environnement `OPENAI_API_KEY`.  
3. Dans **API Gateway**, créez une ressource `/chat` avec la méthode **POST** liée à la Lambda.  
4. Activez le **CORS** pour que votre front‑end Amplify puisse appeler l’API.

### 4. Gestion des quotas API  

| Ressource | Quota par défaut | Astuce d’optimisation |
|-----------|------------------|-----------------------|
| Lambda invocations | 1 000 000/mois | Utilisez **Provisioned Concurrency** uniquement pour les pics (ex. lancement marketing). |
| API Gateway (REST) | 10 000 req/s | Implémentez **caching** côté API (`Cache TTL` 300 s) pour les réponses IA peu changeantes. |
| Bedrock/ SageMaker | 5 000 tokens/mois (Free Tier) | Batch‑process les requêtes hors‑ligne quand possible (ex. génération de FAQ nocturne). |

Surveillez les métriques dans **CloudWatch** et créez des **alarmes** (ex. `Invocations > 80 % du quota`) pour éviter les interruptions de service.

---

## Déployer une solution No‑Code sur Azure Africa  

### 1. Azure Logic Apps + Power Automate  

Azure Logic Apps permet de **chaîner** des connecteurs (Google Sheets, Twilio, OpenAI) sans écrire de code.  

```json
{
  "definition": {
    "$schema": "https://schema.management.azure.com/providers/Microsoft.Logic/schemas/2016-06-01/workflowdefinition.json#",
    "actions": {
      "Appel_OpenAI": {
        "type": "Http",
        "inputs": {
          "method": "POST",
          "uri": "https://api.openai.com/v1/completions",
          "headers": {
            "Authorization": "Bearer @{parameters('openAiKey')}",
            "Content-Type": "application/json"
          },
          "body": {
            "model": "gpt-3.5-turbo",
            "prompt": "@{triggerBody()['text']}",
            "max_tokens": 150
          }
        }
      }
    },
    "triggers": {
      "When_HTTP_Request_Received": {
        "type": "Request",
        "kind": "Http",
        "inputs": {
          "schema": {
            "type": "object",
            "properties": {
              "text": { "type": "string" }
            },
            "required": [ "text" ]
          }
        }
      }
    }
  },
  "parameters": {
    "openAiKey": {
      "type": "SecureString"
    }
  }
}
```

1. Créez la Logic App dans la région **South Africa North**.  
2. Ajoutez le **connector** HTTP ci‑dessus et stockez la clé OpenAI dans **Azure Key Vault**.  
3. Publiez l’URL publique de la Logic App et consommez‑la depuis votre front‑end (Bubble, Softr…).

### 2. Hébergement statique avec Azure Static Web Apps  

```yaml
# staticwebapp.config.json – configuration de routes et d’authentification
{
  "navigationFallback": { "rewrite": "/index.html" },
  "routes": [
    {
      "route": "/api/*",
      "allowedRoles": [ "authenticated" ],
      "backendUri": "https://<logic-app-id>.azurewebsites.net"
    }
  ]
}
```

- Le **pipeline CI/CD** s’appuie sur **GitHub Actions** (déjà intégré).  
- Azure fournit un **CDN Edge** qui possède des points de présence à Johannesburg, améliorant la latence pour les utilisateurs du Kenya ou du Nigeria.

### 3. Contrôle des quotas et optimisation  

| Service | Quota Free | Astuce |
|---------|------------|--------|
| Azure Functions (consommation) | 1 000 000 exécutions/mois | Activez le **plan Premium** uniquement pendant les campagnes publicitaires. |
| Logic Apps | 4 000 actions/mois | Regroupez les appels IA dans un seul **batch** (ex. 10 prompts → 1 appel). |
| Storage (Blob) | 5 GB | Utilisez le **tier Cool** pour les archives de logs IA. |

Utilisez **Azure Monitor** → **Metrics Explorer** pour créer des alertes sur le nombre d’exécutions et le temps de réponse moyen.

---

## Déployer une solution No‑Code sur Google Cloud Africa  

### 1. Firebase Hosting + Cloud Functions  

1. Installez le CLI Firebase (`npm i -g firebase-tools`).  
2. Initialise le projet dans la région **southafrica‑west** :

```bash
firebase init hosting functions
# Choisissez Node.js 18, région southafrica-west
```

3. Implémentez la fonction qui appelle Vertex AI (ou OpenAI) :

```javascript
// functions/index.js
const functions = require('firebase-functions');
const { VertexAI } = require('@google-cloud/vertexai');

const vertex = new VertexAI({ project: process.env.GCLOUD_PROJECT, location: 'southafrica-west' });
const model = vertex.getGenerativeModel({ model: 'gemini-1.5-pro' });

exports.chat = functions.https.onRequest(async (req, res) => {
  const prompt = req.body.prompt;
  const response = await model.generateContent(prompt);
  res.json({ answer: response.text() });
});
```

4. Déployez :

```bash
firebase deploy --only hosting,functions
```

Firebase Hosting fournit un **CDN mondial** avec des points de présence à Johannesburg, garantissant une latence < 100 ms pour les utilisateurs d’Afrique de l’Ouest.

### 2. Gestion des quotas Vertex AI  

- **Free Tier** : 100 k tokens/mois.  
- **Pay‑as‑you‑go** : 0,000 $ par token supplémentaire.  

Pour éviter les dépassements, activez le **budget** dans la console GCP (`Billing → Budgets & alerts`). Configurez une **règle d’arrêt** qui désactive la Cloud Function quand le coût quotidien dépasse un seuil (ex. 5 $).

### 3. Monitoring avec Cloud Operations  

- **Logs** : `gcloud logging read "resource.type=cloud_function"`  
- **Metrics** : `gcloud monitoring dashboards create --config-from-file=dashboard.json`  
- Créez une alerte sur `function/execution_count` > 80 % du quota gratuit.

---

## Hébergement No‑Code dédié : Webflow, Vercel & Netlify  

| Plateforme | Points de présence en Afrique | Avantages No‑Code | Limites |
|------------|------------------------------|-------------------|---------|
| **Webflow** | CDN Cloudflare (POPs à Johannesburg, Nairobi) | Designer visuel complet, CMS intégré | Pas d’accès direct aux fonctions serveur, API limitées |
| **Vercel** | Edge Network incluant **Johannesburg** et **Nairobi** | Déploiement instantané via Git, **Serverless Functions** (Node, Go) | Le plan gratuit impose 100 GB de bande passante/mois |
| **Netlify** | CDN Fastly avec POPs à Cape Town | Build automatisé, fonctions **Netlify Edge** | Les fonctions Edge sont limitées à 50 ms d’exécution |

### Exemple de fonction serverless Vercel (Node)  

```js
// api/chat.js – Vercel Serverless Function
import fetch from 'node-fetch';

export default async function handler(req, res) {
  const { prompt } = req.body;
  const response = await fetch('https://api.openai.com/v1/completions', {
    method: 'POST',
    headers: {
      'Authorization': `Bearer ${process.env.OPENAI_API_KEY}`,
      'Content-Type': 'application/json'
    },
    body: JSON.stringify({
      model: 'gpt-3.5-turbo',
      prompt,
      max_tokens: 150
    })
  });
  const data = await response.json();
  res.status(200).json({ answer: data.choices[0].text.trim() });
}
```

- Ajoutez `OPENAI_API_KEY` dans le **Dashboard → Settings → Environment Variables**.  
- Vercel crée automatiquement un **endpoint** `https://your-project.vercel.app/api/chat`.  

Ces plateformes offrent un **déploiement “one‑click”** idéal pour les MVP africains qui ne nécessitent pas de configuration réseau complexe.

---

## Sécuriser les données sensibles  

### 1. Conformité RGPD & législations africaines  

| Pays | Loi principale | Points clés |
|------|----------------|-------------|
| Kenya | **Data Protection Act 2019** | Consentement explicite, droit à l’effacement, localisation des données recommandée. |
| Nigeria | **Nigeria Data Protection Regulation (NDPR)** | Notification de violation sous 72 h, chiffrement au repos obligatoire. |
| Afrique du Sud | **POPIA** | Classification des données (public, privé), chiffrement AES‑256 recommandé. |

**Bonnes pratiques universelles** :

- **Chiffrez** les données au repos (S3 AES‑256, Azure Blob Encryption, Cloud SQL TDE).  
- **TLS 1.2+** obligatoire pour toutes les communications (HTTPS, wss).  
- Stockez les secrets dans **AWS Secrets Manager**, **Azure Key Vault** ou **Google Secret Manager** – jamais en clair dans le code.  
- Implémentez le **principle of least privilege** (IAM) : chaque service ne possède que les permissions strictement nécessaires.

### 2. Gestion des accès utilisateur  

- Utilisez **Cognito** (AWS), **Azure AD B2C** ou **Firebase Auth** pour l’authentification.  
- Activez **MFA** (SMS ou authentificateur) pour les comptes administrateurs.  
- Implémentez **RBAC** (rôles : admin, manager, client) au niveau de l’API (ex. via API Gateway authorizer ou Azure AD claims).

### 3. Audits et journalisation