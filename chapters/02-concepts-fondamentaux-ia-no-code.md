## Qu’est‑ce que l’intelligence artificielle ?  

L’intelligence artificielle (IA) désigne tout système capable d’accomplir, de façon autonome, des tâches qui requièrent habituellement l’intelligence humaine : compréhension du langage, reconnaissance d’images, prise de décision, etc. En Afrique, l’IA se manifeste souvent dans des contextes où l’accès à la technologie est limité : un petit commerçant qui veut automatiser la classification de ses factures, un agriculteur qui veut anticiper la météo locale, ou encore un service de santé qui doit trier rapidement les symptômes signalés par les patients via WhatsApp.  

Contrairement à l’automatisation « classique », qui repose sur des règles fixes (ex. : « si le montant > 1000 €, appliquer 10 % de remise »), l’IA utilise des modèles statistiques qui apprennent à partir de données. Cette capacité d’adaptation est le moteur qui rend possible la création de solutions personnalisées sans écrire une seule ligne de code.

---

## Apprentissage automatique : les bases  

### Apprentissage supervisé  

Dans l’apprentissage supervisé, le modèle est entraîné à partir d’un jeu de données **étiquetées** : chaque exemple d’entrée est associé à la réponse attendue. Par exemple, pour classer des produits agricoles (maïs, manioc, cacao) à partir de leurs descriptions, on fournit à l’algorithme des dizaines de milliers de descriptions déjà classées. Le modèle apprend alors les motifs qui différencient chaque catégorie et peut, à l’avenir, attribuer automatiquement une catégorie à une nouvelle description.  

Les tâches typiques :  
* **Classification** (texte, image) – ex. : reconnaître si un commentaire client est positif ou négatif.  
* **Régression** – ex. : prédire le prix d’un sac de riz en fonction de la saison et du volume commandé.  

### Apprentissage non‑supervisé  

Ici, aucune étiquette n’est fournie ; le modèle doit **découvrir** des structures cachées dans les données. Les algorithmes de clustering (k‑means, DBSCAN) regroupent les points similaires, tandis que les techniques de réduction de dimension (PCA, t‑SNE) permettent de visualiser des jeux de données complexes.  

Exemple concret : un réseau de distribution d’eau veut segmenter ses usagers en fonction de leurs habitudes de consommation, sans disposer d’étiquettes pré‑définies. Un clustering révèle trois groupes : « ménages à faible consommation », « petites entreprises », et « grandes exploitations ». Ces groupes servent ensuite à ajuster les tarifs ou à cibler des campagnes de sensibilisation.  

### Exemple pratique : classification de produits agricoles  

| Description du produit | Étiquette attendue |
|------------------------|--------------------|
| « Grains de maïs blanc, 25 kg, origine Bénin » | Maïs |
| « Racines de manioc séchées, 10 kg, Côte d’Ivoire » | Manioc |
| « Fèves de cacao fermentées, 5 kg, Ghana » | Cacao |

Un modèle supervisé pré‑entraîné (ex. : `distilbert-base-fr-cased` de Hugging Face) peut être **fine‑tuned** avec ces quelques centaines d’exemples, puis intégré à une plateforme no‑code pour classer automatiquement chaque nouveau produit ajouté à un catalogue en ligne.

---

## Modèles pré‑entraînés et APIs IA  

### Pourquoi les modèles pré‑entraînés  

Construire un modèle à partir de zéro nécessite des dizaines de milliers d’exemples, du temps de calcul et une expertise en data‑science. Les modèles pré‑entraînés offrent :  

* **Un point de départ** : ils ont déjà appris des représentations générales du langage ou des images.  
* **Une réduction du coût** : le calcul intensif est réalisé une fois par le fournisseur (OpenAI, Hugging Face, Cohere).  
* **Une adaptation rapide** : grâce au fine‑tuning ou aux prompts, on peut les spécialiser à un problème local (par ex. : français africain, termes de l’agriculture de la région du Sahel).  

### Principaux fournisseurs d’APIs  

| Fournisseur | Types de modèles | Points forts pour l’Afrique francophone |
|-------------|------------------|------------------------------------------|
| **OpenAI**  | GPT‑4, embeddings, DALL‑E | Excellente maîtrise du français, documentation claire, tarification à l’usage. |
| **Hugging Face** | Transformers (BERT, RoBERTa, Whisper) | Large bibliothèque open‑source, possibilité d’héberger un modèle sur un serveur local (ex. : sur un VPS au Nigeria). |
| **Cohere**  | Command‑R, embeddings | Tarifs compétitifs pour les volumes élevés, support multilingue. |

Tous ces services exposent leurs capacités via des **endpoints HTTP**. Une plateforme no‑code peut appeler ces endpoints grâce à un bloc *Webhooks* ou *API Connector*.  

### Exemple d’appel API en no‑code (Zapier Webhooks)  

Supposons que l’on veuille analyser le sentiment d’un message WhatsApp reçu via Twilio et stocker le résultat dans Google Sheets. Le flux Zapier ressemble à :  

1. **Trigger** : « New Incoming Message » (Twilio).  
2. **Action** : « Webhooks by Zapier » → **POST** vers l’API OpenAI.  

```json
{
  "model": "gpt-4o-mini",
  "messages": [
    {"role": "system", "content": "Analyse le sentiment du texte suivant (positif, neutre, négatif)."},
    {"role": "user", "content": "{{MessageBody}}"}
  ],
  "temperature": 0
}
```

3. **Action** : extraire le champ `choices[0].message.content` de la réponse et l’insérer dans une ligne Google Sheets.  

Ce même principe s’applique avec **Make** (ancien Integromat) ou **Parabola** : il suffit de fournir l’URL, les headers (`Authorization: Bearer <clé>`), et le corps JSON. Aucun code n’est requis, mais la logique sous‑jacente reste la même que dans un script Python traditionnel.

---

## Fonctionnement des plateformes no‑code IA  

### Architecture typique  

1. **Interface utilisateur** : éditeur drag‑and‑drop (Bubble, Softr, Adalo) où l’on crée les pages, formulaires et tableaux de bord.  
2. **Moteur de workflow** : moteur d’automatisation (Zapier, Make, n8n) qui orchestre les déclencheurs, les conditions et les actions.  
3. **Connecteurs** : modules prêts à l’emploi pour appeler des APIs externes, manipuler des bases de données, ou invoquer des modèles IA.  

Cette séparation permet à un entrepreneur de modifier l’UX sans toucher à la logique IA, et inversement.  

### Types de blocs IA dans les constructeurs  

| Bloc | Fonction | Exemple d’usage |
|------|----------|-----------------|
| **Prompt Builder** | Assemble un texte dynamique à envoyer à un LLM | Générer une description de produit à partir de ses caractéristiques. |
| **Embedding Generator** | Convertit un texte en vecteur numérique | Recherche sémantique de documents juridiques en droit foncier. |
| **Image Generation** | Crée une image à partir d’une description | Produire des visuels publicitaires pour une campagne de micro‑finance. |
| **Classification** | Retourne une catégorie prédéfinie | Filtrer automatiquement les messages de spam sur une plateforme de e‑learning. |

### Exemple : formulaire d’avis client avec Softr + OpenAI  

1. **Création du formulaire** : champs « Nom », « Commentaire ».  
2. **Déclencheur** : « When a new record is created » (Softr → Webhook).  
3. **Action IA** : appel à l’API OpenAI avec le prompt :  

```
Tu es un analyste de sentiment. Classe le commentaire suivant comme Positif, Neutre ou Négatif.
Commentaire : "{{Commentaire}}"
```  

4. **Mise à jour du record** : ajouter le champ « Sentiment » avec la réponse de l’IA.  
5. **Affichage** : tableau filtré par sentiment, permettant au manager de repérer rapidement les retours négatifs à traiter.

---

## Limites et points forts du no‑code IA  

### Avantages  

| Avantage | Illustration concrète |
|----------|-----------------------|
| **Rapidité de mise sur le marché** | Un agriculteur peut lancer un chatbot de conseils en moins de 48 h. |
| **Coût initial réduit** | Pas besoin d’embaucher un data‑scientist ; le tarif d’une API est souvent inférieur à celui d’un développeur senior. |
| **Accessibilité** | Les entrepreneurs qui maîtrisent Excel ou Google Sheets peuvent désormais exploiter l’IA sans formation technique. |
| **Itération facile** | Modifier le prompt ou le flux d’automatisation se fait en quelques clics, idéal pour tester plusieurs hypothèses. |

### Contraintes  

| Limite | Conséquence | Contournement possible |
|--------|-------------|------------------------|
| **Personnalisation limitée** | Les modèles pré‑entraînés ne comprennent pas toujours les termes locaux (ex. : « sorgho », « brouette »). | Utiliser le *fine‑tuning* sur un petit jeu de données local, ou combiner plusieurs prompts. |
| **Scalabilité** | Un grand nombre de requêtes peut entraîner des latences ou des coûts élevés. | Mettre en place un cache (ex. : stocker les réponses fréquentes dans Redis) ou migrer vers un modèle hébergé en interne. |
| **Gouvernance des données** | Les données sensibles (données de santé, informations financières) sont envoyées à des serveurs externes. | Choisir un fournisseur proposant le **data residency** en Afrique ou héberger un modèle open‑source sur un serveur local. |
| **Verrouillage propriétaire** | Dépendance à une plateforme qui peut changer ses tarifs ou ses APIs. | Concevoir le workflow de façon modulaire : chaque appel IA passe par un **wrapper** (ex. : un petit micro‑service sur Railway) qui peut être redirigé vers un autre fournisseur. |

### Astuce : le modèle hybride low‑code  

Lorsque le besoin dépasse les capacités d’un outil no‑code (ex. : entraînement d’un modèle à partir de plusieurs millions d’enregistrements), il est judicieux d’ajouter un petit composant de code (Node.js sur Vercel, Python sur Render) qui réalise le traitement lourd, puis de le déclencher depuis le workflow no‑code. Cette approche conserve la rapidité du no‑code tout en offrant la flexibilité du code sur mesure.

---

## Choisir le bon modèle IA selon le problème métier  

### Cartographie des besoins  

| Domaine | Type de donnée | Modèle recommandé | Exemple d’application en Afrique |
|---------|----------------|-------------------|-----------------------------------|
| **Texte** | Analyse de sentiment, génération de réponses | GPT‑4o-mini, Claude 3 Haiku | Chatbot d’assistance client sur WhatsApp pour une boutique de tissus. |
| **Image** | Classification d’objets, génération d’images | CLIP, Stable Diffusion | Identifier les maladies du maïs à partir de photos prises avec un smartphone. |
| **Audio** | Transcription, reconnaissance vocale | Whisper, Vosk | Convertir les appels téléphoniques d’un centre de santé en texte pour archivage. |
| **Prédiction** | Séries temporelles, régression | Prophet, XGBoost via API | Prévoir la demande de carburant dans une station-service de Bamako. |

### Critères de sélection  

1. **Langue et dialecte** : privilégier les modèles entraînés sur du français et, si possible, sur des corpus africains (ex. : `camembert-base-fra-afrique`).  
2. **Latence** : les applications en temps réel (chatbot, assistance vocale) nécessitent des réponses < 300 ms ; choisir un fournisseur avec des **edge locations** proches (ex. : Azure Africa Central).  
3. **Coût par appel** : calculer le volume mensuel prévu (ex. : 10 000 requêtes texte à 0,0005 $ = 5 $).  
4. **Capacité de fine‑tuning** : si le domaine est très spécifique (ex. : termes juridiques du droit foncier nigérian), opter pour un modèle qui autorise le fine‑tuning.  
5. **Conformité** : vérifier que le fournisseur respecte les régulations locales (ex. : GDPR‑like lois au Maroc).  

### Cas d’usage africain : prédire la demande de maïs à partir de données météo  

1. **Collecte des données** : historiques de ventes (Google Sheets), prévisions météo (API OpenWeather).  
2. **Pré‑traitement** : agrégation hebdomadaire, création de variables « précipitations », « température moyenne ».  
3. **Modèle** : `prophet` (Facebook Prophet) exposé via une API Python hébergée sur **Render**.  
4. **Intégration no‑code** :  
   * **Trigger** : chaque dimanche, le workflow