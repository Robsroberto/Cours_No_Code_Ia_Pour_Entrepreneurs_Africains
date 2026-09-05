## Préparer les données client dans Google Sheets  

Avant de pouvoir exploiter l’intelligence artificielle, il faut que vos données soient propres, structurées et accessibles.  

| Colonne | Exemple | Format recommandé |
|---------|---------|-------------------|
| `ClientID` | 10234 | texte ou nombre unique |
| `Nom` | **Aïcha Diop** | texte |
| `Pays` | **Sénégal** | texte (liste déroulante) |
| `DateInscription` | 2023‑03‑15 | date ISO (YYYY‑MM‑DD) |
| `DernièreCommande` | 2024‑06‑01 | date |
| `MontantTotal` | 1250.50 | nombre décimal |
| `NbCommandes` | 7 | entier |
| `CanalAcquisition` | **WhatsApp** | texte (enum) |
| `Statut` | **Actif** / **Inactif** | texte |

1. **Uniformiser les formats** : utilisez le format de cellule Google Sheets (`Format > Nombre`) pour les dates et les nombres.  
2. **Éliminer les doublons** : `Données > Nettoyer > Supprimer les doublons`.  
3. **Enrichir les colonnes** : ajoutez des champs calculés (ex. : `JoursDepuisDernièreCommande = AUJOURDHUI() - DernièreCommande`).  

Ces étapes garantissent que le modèle IA recevra des entrées cohérentes, ce qui améliore la qualité des prédictions.

---

## Connecter Google Sheets à un modèle IA sans écrire de code  

### 1. Choisir la passerelle d’automatisation  

| Plateforme | Points forts pour l’Afrique | Exemple d’usage |
|------------|----------------------------|-----------------|
| **Zapier** | Large catalogue d’apps, connexion directe à l’API OpenAI, serveur en Europe mais latence acceptable | Envoi d’un lot de 100 lignes à ChatGPT pour classification |
| **Make (Integromat)** | Scénarios visuels, support de fonctions HTTP avancées, tarification adaptée aux petites structures | Appel à l’API Vertex AI de Google Cloud |
| **Automate.io** (maintenant intégré à Zapier) | Simplicité d’interface, bon pour les PME | Enrichissement de contacts avec des tags IA |

Nous illustrerons le flux avec **Zapier**, mais le même principe s’applique à Make.

### 2. Créer le Zap de base  

1. **Trigger** – *Google Sheets > New or Updated Spreadsheet Row*  
   - Sélectionnez le fichier et la feuille contenant vos clients.  
   - Activez l’option “Trigger on new rows only” pour éviter les boucles.  

2. **Action** – *Webhooks by Zapier > Custom Request*  
   - Méthode : `POST`  
   - URL : `https://api.openai.com/v1/chat/completions` (ou l’endpoint de votre modèle).  
   - Headers :  
     ```json
     {
       "Content-Type": "application/json",
       "Authorization": "Bearer {{YOUR_OPENAI_API_KEY}}"
     }
     ```  
   - Payload :  
     ```json
     {
       "model": "gpt-4o-mini",
       "messages": [
         {
           "role": "system",
           "content": "Tu es un analyste marketing. Analyse les données suivantes et renvoie un JSON contenant les champs demandés."
         },
         {
           "role": "user",
           "content": "ClientID: {{ClientID}}, Pays: {{Pays}}, MontantTotal: {{MontantTotal}}, NbCommandes: {{NbCommandes}}, JoursDepuisDernièreCommande: {{JoursDepuisDernièreCommande}}"
         }
       ],
       "temperature": 0
     }
     ```  

3. **Action** – *Google Sheets > Update Spreadsheet Row*  
   - Mappez la réponse JSON de l’IA (ex. : `segment`, `probabilité_churn`, `produit_rentable`) vers de nouvelles colonnes `SegmentIA`, `ChurnScore`, `ProduitTop`.  

Ce Zap s’exécute à chaque ajout ou modification d’une ligne, envoie les variables à l’IA, puis inscrit les insights directement dans la même feuille.

---

## Segmentation automatisée des clients  

### Objectif  

Classer chaque client dans un segment : **VIP**, **Régulier**, **À risque**, **Nouveau**.  

### Prompt IA recommandé  

```text
Tu es un analyste de données. En te basant sur les champs suivants : MontantTotal, NbCommandes, JoursDepuisDernièreCommande, CanalAcquisition, Statut, attribue le client à l’un des segments suivants :
- VIP : dépenses > 2000 €, >10 commandes, actif depuis > 1 an.
- Régulier : dépenses entre 500 € et 2000 €, 3‑10 commandes.
- À risque : aucune commande depuis plus de 90 jours.
- Nouveau : inscrit depuis moins de 30 jours.
Retourne uniquement le nom du segment sous forme de texte.
```

### Implémentation concrète  

Dans le Zap, le champ `content` du message *user* inclut les variables nécessaires. L’IA renvoie par exemple : `"VIP"`. Zapier écrit ce résultat dans la colonne `SegmentIA`.  

**Cas d’usage africain** : un commerçant de Lagos utilise cette segmentation pour déclencher automatiquement des campagnes WhatsApp ciblées via Twilio (Zapier → Twilio → message).  

---

## Prédire le churn (attrition) des abonnés  

### Pourquoi le churn est crucial  

Dans les services de micro‑finance ou les abonnements SaaS, anticiper les désabonnements permet d’intervenir à temps (offre de fidélisation, appel commercial).  

### Modèle de prédiction simplifié  

Au lieu de former un modèle complexe, on exploite **ChatGPT** comme classificateur : on lui fournit les indicateurs clés et on lui demande de renvoyer une probabilité de churn (0‑100).  

#### Prompt IA pour le churn  

```text
Analyse les données suivantes d’un client et estime la probabilité qu’il se désabonne dans les 30 prochains jours (0 = pas de risque, 100 = risque certain). Retourne uniquement un nombre entier.
ClientID: {{ClientID}}
MontantTotal: {{MontantTotal}}
NbCommandes: {{NbCommandes}}
JoursDepuisDernièreCommande: {{JoursDepuisDernièreCommande}}
Statut: {{Statut}}
CanalAcquisition: {{CanalAcquisition}}
```

#### Traitement de la réponse  

- Si la valeur ≥ 70, le champ `ChurnScore` reçoit `"Haut"` et le Zap déclenche une **action de notification** (Slack, WhatsApp).  
- Si la valeur < 30, on considère le client « sain ».  

### Exemple de flux complet  

| Étape | Action |
|-------|--------|
| 1 | Trigger sur mise à jour de la ligne (nouvelle commande). |
| 2 | Webhook → IA (prompt churn). |
| 3 | Parse JSON → `churn_probability`. |
| 4 | Condition Zapier : `churn_probability >= 70` ? |
| 5a | Oui → Envoi d’un SMS via Twilio à l’équipe de rétention. |
| 5b | Non → Mise à jour de la colonne `ChurnScore`. |

Ce processus fonctionne même avec une connexion mobile 3G/4G, car les appels API sont légers (quelques kilooctets).

---

## Identifier les produits les plus rentables  

### Méthode de scoring produit  

On agrège les ventes par SKU, puis on demande à l’IA de classer les produits selon la marge brute et la fréquence d’achat.  

#### Étape 1 : agrégation dans Google Sheets  

Utilisez la fonction `QUERY` :  

```gs
=QUERY(A2:G, "SELECT G, SUM(E) AS TotalVentes, COUNT(F) AS NbVentes GROUP BY G ORDER BY TotalVentes DESC", 1)
```  

- `G` = colonne `SKU`  
- `E` = `MontantTotal` (par ligne)  
- `F` = `NbCommandes`  

Cette table résumée (appelée *TableauProduit*) sert de source d’entrée pour l’IA.

#### Étape 2 : appel IA via Zapier (ou Make)  

- Trigger : nouvelle ligne dans *TableauProduit*.  
- Prompt :  

```text
Voici les ventes d’un produit : SKU {{SKU}}, total ventes {{TotalVentes}} €, nombre de ventes {{NbVentes}}. Classe ce produit parmi les catégories suivantes : "Très rentable", "Rentable", "Moyen", "Faible". Retourne uniquement le libellé.
```  

- Résultat inscrit dans la colonne `Rentabilité`.  

#### Utilisation opérationnelle  

Le commerçant de Kigali crée un tableau de bord simple (Google Data Studio) qui filtre les produits `Très rentable`. Il peut alors augmenter les stocks ou lancer des promotions ciblées.

---

## Visualiser les insights directement dans Google Sheets  

### Mise en forme conditionnelle  

| Colonne | Règle de mise en forme | Couleur |
|---------|------------------------|---------|
| `SegmentIA` | `"VIP"` | Vert foncé |
| `SegmentIA` | `"À risque"` | Rouge |
| `ChurnScore` | `>= 70` | Fond orange |
| `Rentabilité` | `"Très rentable"` | Fond bleu |

Accédez à *Format > Mise en forme conditionnelle* et créez les règles ci‑dessus. Les indicateurs deviennent immédiatement lisibles.

### Graphiques dynamiques  

- **Histogramme** : nombre de clients par segment.  
- **Courbe** : évolution du `ChurnScore` moyen par mois.  
- **Camembert** : part de chaque catégorie de rentabilité.  

Ces graphiques se rafraîchissent automatiquement dès que le Zap met à jour les lignes.

---

## Créer une application no‑code avec AppSheet  

AppSheet (Google) permet de transformer votre feuille en une application mobile/web sans écrire de code.  

### Étapes clés  

1. **Connexion** : dans AppSheet, choisissez “Start with your own data” et sélectionnez le fichier Google Sheets contenant les colonnes `SegmentIA`, `ChurnScore`, etc.  
2. **Définir les vues** :  
   - *Clients* : tableau avec filtres par segment.  
   - *Alertes churn* : vue filtrée `ChurnScore = "Haut"` avec bouton “Notifier l’équipe”.  
   - *Produits rentables* : galerie d’images des SKU classés “Très rentable”.  
3. **Actions personnalisées** : ajoutez un bouton “Envoyer SMS” qui appelle le service Twilio via le connecteur intégré d’AppSheet.  
4. **Règles de sécurité** : restreignez l’accès aux données sensibles (ex. : uniquement les managers peuvent voir les scores de churn).  

L’application se synchronise en temps réel avec la feuille ; chaque mise à jour IA apparaît immédiatement sur le mobile des commerciaux, même dans les zones rurales où la connectivité est intermittente.

---

## Bonnes pratiques et limites à connaître  

| Aspect | Recommandation |
|--------|----------------|
| **Volume de requêtes IA** | Les plans gratuits d’OpenAI ou de Vertex AI imposent des quotas (ex. : 100 000 tokens/mois). Planifiez des batchs (ex. : 50 lignes à la fois) pour éviter les dépassements. |
| **Confidentialité des données** | Anonymisez les champs sensibles (numéro de téléphone, email) avant l’envoi à l’API. Utilisez la variable `{{DataMask}}` ou créez une colonne `ClientHash` (SHA‑256). |
| **Latence** | Un appel API prend généralement 500 ms – 2 s. Si vous avez besoin de réponses quasi‑instantanées, limitez le nombre de colonnes transmises. |
| **Qualité du prompt** | Plus le prompt est précis, plus la réponse est fiable. Testez plusieurs variantes dans l’onglet “Playground” d’OpenAI avant de le mettre en production. |
| **Gestion des erreurs** | Dans Zapier, ajoutez une étape “Filter” qui vérifie que la réponse contient bien le champ attendu; sinon, redirigez vers un tableau d’erreurs pour analyse. |
| **Mise à jour incrémentale** | Utilisez une colonne `DernièreAnalyse` (date‑heure) pour ne traiter que les lignes modifiées depuis la dernière exécution du Zap. |
| **Évolution du modèle** | Les modèles pré‑entraînés évoluent (nouveaux modèles, nouvelles capacités). Ré‑évaluez vos prompts tous les 3‑6 mois pour profiter des améliorations. |

---

## Points clés  

- **Structuration préalable** : des données propres et normalisées sont la base d’une IA fiable.  
- **Zapier/Make** offrent des passerelles no‑code pour appeler des modèles IA (OpenAI, Vertex AI) depuis Google Sheets.  
- **Segmentation, churn et rentabilité** peuvent être automatisés via des prompts ciblés et les résultats écrits directement dans la feuille.  
- **Mise en forme conditionnelle et graphiques** transforment les réponses IA en visualisations actionnables.  
- **AppSheet** convertit la feuille enrichie en application mobile, permettant aux équipes terrain d’accéder aux insights même hors ligne.  
- **Surveiller les quotas, sécuriser les données et itérer les prompts** sont essentiels pour une solution durable.  

En appliquant ces étapes, tout entrepreneur africain peut exploiter la puissance de l’intelligence artificielle sans écrire une seule ligne de code, tout en restant maître de ses données et de ses processus métier.