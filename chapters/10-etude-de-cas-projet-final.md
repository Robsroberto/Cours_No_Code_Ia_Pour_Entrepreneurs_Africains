## Étude de cas : Kofi, agritech au Ghana  

### Contexte et problème  

Kofi Mensah, agriculteur‑entrepreneur de la région d’Ashanti, gère une coopérative de 120 petits exploitants de maïs et de manioc. Avant 2022, la coopérative faisait face à trois difficultés majeures :  

1. **Variabilité des rendements** – les prévisions étaient basées sur l’expérience du chef de culture, ce qui entraînait des écarts de ±30 % entre les prévisions et les récoltes réelles.  
2. **Manque d’accès à l’expertise agronome** – les agriculteurs n’avaient pas de conseiller disponible en temps réel pour poser des questions sur les traitements phytosanitaires ou les dates de semis.  
3. **Gestion manuelle des stocks** – les entrées et sorties de semences, engrais et produits phytosanitaires étaient saisies dans un classeur Excel partagé, source fréquente d’erreurs et de pertes de temps.  

Kofi a décidé de transformer ces points de friction en opportunités grâce aux outils No‑Code IA, afin de créer une plateforme « SmartFarm » capable de :  

- prédire le rendement de chaque parcelle à partir de données climatiques et de pratiques culturales,  
- fournir un chatbot agronome disponible sur WhatsApp,  
- automatiser la mise à jour des stocks et l’envoi de rappels de réapprovisionnement.  

### Construction de la solution No‑Code IA  

#### Choix des outils  

| Besoin | Outil choisi | Raison du choix (Afrique) |
|--------|--------------|---------------------------|
| Interface web & base de données | **Bubble** | Hébergement en Europe avec CDN en Afrique, aucune ligne de code, support multilingue. |
| Automatisation des flux | **Zapier** (plan Starter) | Large catalogue de connecteurs (Google Sheets, WhatsApp Business API via Twilio), tarification adaptée aux petites structures. |
| Stockage & calculs simples | **Google Sheets** | Accessibilité hors ligne grâce à l’app mobile, familiarité des agriculteurs avec les tableurs. |
| Modélisation IA (prédiction rendement) | **OpenAI GPT‑4 via API** (prompt engineering) + **Parabola** pour le pré‑traitement des données | API fiable, facturation à la consommation, possibilité d’utiliser des prompts pour créer un modèle de régression sans écrire de code. |
| Chatbot WhatsApp | **Landbot** + **OpenAI** | Interface drag‑and‑drop, connexion directe à l’API WhatsApp Business via Twilio, réponses naturelles grâce à GPT‑4. |

> Voir chapitre 3 pour la création de comptes et la connexion des API.

#### Création du tableau de données des cultures  

Kofi a structuré les informations essentielles dans une feuille Google Sheets nommée **« Data_Farm »** :  

| Colonne | Description |
|---------|-------------|
| `parcel_id` | Identifiant unique de la parcelle |
| `crop_type` | Maïs / Manioc |
| `sowing_date` | Date de semis |
| `area_ha` | Surface en hectares |
| `fertilizer_kg` | Quantité d’engrais appliquée |
| `rainfall_mm` | Cumul des précipitations (30 jours) |
| `temperature_c` | Température moyenne (30 jours) |
| `yield_kg` | Rendement réel (rempli après récolte) |

Les agriculteurs remplissent les colonnes **avant** la saison via un formulaire Bubble intégré à la feuille (voir chapitre 8).  

#### Modélisation IA pour prédiction de rendement  

Au lieu d’entraîner un modèle de machine learning classique, Kofi a utilisé le **prompt engineering** : il envoie les variables de chaque parcelle à l’API OpenAI, qui renvoie une estimation du rendement. Le prompt utilisé est stocké dans **Parabola** :

```json
{
  "model": "gpt-4",
  "messages": [
    {"role": "system", "content": "You are an agronomy expert specialized in Ghanaian climate."},
    {"role": "user", "content": "Predict the maize yield (kg/ha) for a parcel with the following data: area=1.5 ha, fertilizer=120 kg, rainfall=250 mm, temperature=27°C, sowing_date=2023-04-15."}
  ],
  "temperature": 0.0,
  "max_tokens": 50
}
```

Parabola récupère les lignes de **Data_Farm**, construit dynamiquement le prompt pour chaque parcelle, appelle l’API et écrit la réponse dans la colonne `predicted_yield_kg`.  

#### Intégration du chatbot pour assistance agronome  

Le chatbot Landbot est configuré avec trois **blocks** principaux :  

1. **Accueil** – demande le type de culture (`crop_type`).  
2. **Question** – transmet la requête de l’utilisateur à l’API OpenAI avec le contexte « agronomie Ghana ».  
3. **Réponse** – renvoie le texte généré à l’utilisateur sur WhatsApp.  

Un exemple de **payload** envoyé depuis Landbot à OpenAI :

```json
{
  "model": "gpt-4",
  "messages": [
    {"role": "system", "content": "You are a friendly agronomist helping Ghanaian farmers. Answer concisely."},
    {"role": "user", "content": "What is the optimal time to apply fertilizer for maize in Ashanti region?"}
  ],
  "temperature": 0.3,
  "max_tokens": 120
}
```

Le flux Zapier relie Landbot → Google Sheets (enregistrement de chaque interaction) → Slack (alerte à Kofi lorsqu’une question critique est posée).  

### Défis rencontrés et comment les surmonter  

| Défi | Solution appliquée |
|------|--------------------|
| **Accès limité à Internet** (zones rurales) | Utilisation de **Google Sheets offline** et d’un **cache local** sur le téléphone via l’app Bubble Progressive Web App (PWA). Les données sont synchronisées dès qu’une connexion est disponible. |
| **Qualité des données** (valeurs manquantes, fautes de frappe) | Mise en place de **règles de validation** dans le formulaire Bubble (ex. : champ `rainfall_mm` > 0). Parabola ajoute un **step de nettoyage** : remplacement des valeurs vides par la moyenne régionale. |
| **Coût des appels API** (OpenAI) | Kofi a limité les appels aux **prévisions** à la fin de chaque semaine (batch) et a activé le **mode “temperature = 0”** pour obtenir des réponses déterministes, réduisant ainsi le nombre de tokens consommés. Un budget mensuel de 30 USD a été fixé, surveillé via le tableau de bord Zapier. |
| **Intégration WhatsApp** (numéro Business) | Utilisation de **Twilio Sandbox** pendant la phase pilote, puis passage à un numéro local via le partenaire télécom de Ghana, ce qui a réduit les frais de messages entrants de 70 %. |

### Résultats obtenus  

- **Précision des prévisions** : l’écart moyen entre le rendement prédit et le rendement réel est passé de ±30 % à **±8 %** après trois cycles de saison.  
- **Adoption du chatbot** : plus de **2 500** messages échangés en six mois, avec un taux de satisfaction de 92 % (questionnaire intégré dans Landbot).  
- **Gain de temps administratif** : les agriculteurs déclarent une réduction de 4 heures par semaine dans la tenue de leurs registres, grâce à la saisie automatisée via le formulaire Bubble.  
- **Impact économique** : la coopérative a augmenté son chiffre d’affaires de **35 %** en 2023, principalement grâce à une meilleure planification des intrants et à la réduction des pertes post‑récolte.  

Ces résultats ont permis à Kofi de lever **US $50 000** auprès d’un fonds d’impact africain, qui finance désormais l’extension du service à deux nouvelles régions du pays.  

---

## Leçons à retenir de la success‑story  

### Méthodologie itérative  

Kofi a adopté le principe du **MVP (Produit Minimum Viable)** : il a d’abord construit le tableau de prévision et le chatbot basique, puis a ajouté les automatisations de stock et les alertes. Chaque itération était testée auprès d’un petit groupe d’agriculteurs avant d’être déployée à l’ensemble de la coopérative.  

### Importance du partenariat local  

Le succès repose sur la collaboration avec :  

- **Un fournisseur d’accès mobile** qui a offert des forfaits data à prix réduit aux membres,  
- **Un centre de formation agricole** qui a animé des ateliers de prise en main du formulaire Bubble,  
- **Un développeur freelance** (local) qui a aidé à configurer le webhook Twilio.  

Ces partenariats ont réduit les frictions d’adoption et ont renforcé la confiance dans la solution.  

### Gestion du budget IA  

En limitant les appels API à des **batchs hebdomadaires**, en utilisant des **prompts deterministes** et en surveillant les métriques de consommation via le tableau de bord Zapier, Kofi a pu garder les coûts IA sous le seuil de 30 USD/mois, un montant réaliste pour une petite coopérative.  

---

## Concevoir votre projet final  

### Définir le problème et les objectifs SMART  

1. **Spécifique** – Quel processus voulez‑vous optimiser ? (ex. : réduction du temps de facturation).  
2. **Mesurable** – Quel KPI suivrez‑vous ? (ex. : minutes économisées par facture).  
3. **Atteignable** – Les données nécessaires sont‑elles disponibles ?  
4. **Pertinent** – L’objectif apporte‑t‑il une vraie valeur ajoutée à votre business ?  
5. **Temporel** – Quelle est la date cible pour le MVP ?  

### Cartographier le flux de travail No‑Code  

```
[Formulaire Web (Bubble)] → [Google Sheets] → [Parabola (pré‑traitement)] → 
[OpenAI API (prédiction / génération)] → [Zapier] → 
   ├─► [Email (résultat) via Gmail]  
   └─► [Notification WhatsApp via Twilio]
```

Chaque bloc représente une étape que vous devez configurer dans la plateforme correspondante.  

### Sélectionner la stack d’outils adaptée  

| Fonction | Outil recommandé | Pourquoi |
|----------|------------------|----------|
| Interface client | **Bubble** | Rapide à prototyper, responsive, support mobile. |
| Automatisation | **Zapier** ou **Make** | Large choix de connecteurs, interface visuelle. |
| Stockage léger | **Google Sheets** | Collaboration en temps réel, accessible hors ligne. |
| IA texte | **OpenAI GPT‑4** | Modèles performants, facturation à la demande. |
| IA image (si besoin) | **DALL·E** ou **Midjourney** | Génération d’illustrations marketing. |
| Chatbot multi‑canal | **Landbot** + **Twilio** | Déploiement sur web, WhatsApp, Facebook Messenger. |

> Voir chapitre 5 pour les bonnes pratiques de connexion Zapier‑OpenAI.  

### Prototyper rapidement (MVP)  

1. **Créer le formulaire** dans Bubble (voir chapitre 8).  
2. **Connecter le formulaire à Google Sheets** via le plugin natif Bubble → Sheets.  
3. **Construire le flux Parabola** : importer les lignes, appliquer une fonction `IF` pour remplacer les valeurs manquantes, concaténer les champs dans un prompt, appeler l’API OpenAI, écrire le résultat.  

#### Exemple de script Zapier (JSON) pour déclencher l’IA dès qu’une ligne est ajoutée  

```json
{
  "trigger": {
    "type": "google_sheet_new_row",
    "sheet_id": "1aBcDeFgHiJkLmNoPqRsTuVwXyZ",
    "worksheet_name": "Data_Farm"
  },
  "actions": [
    {
      "type": "http_request",
      "method": "POST",
      "url": "https://api.openai.com/v1/chat/completions",
      "headers": {
        "Authorization": "Bearer {{zapier_secret_openai_key}}",
        "Content-Type": "application/json"
      },
      "body": {
        "model": "gpt-4",
        "messages": [
          {"role": "system", "content": "You are an agronomy expert for Ghana."},
          {"role": "user", "content": "Predict yield for parcel {{parcel_id}}: area={{area_ha}}ha, fertilizer={{fertilizer_kg}}kg, rainfall={{rainfall_mm}}mm, temperature={{temperature_c}}°C."}
        ],
        "temperature": 0,
        "max_tokens": 60
      }
    },
    {
      "type": "google_sheet_update_row",
      "sheet_id": "1aBcDeFgHiJkLmNoPqRsTuVwXyZ",
      "worksheet_name": "Data_Farm",
      "row_id": "{{row_id}}",
      "fields": {
        "predicted_yield_kg": "{{response.choices[0].message.content}}"
      }
    }
  ]
}
```

Ce Zap crée une **prévision instantanée** chaque fois qu’un agriculteur saisit une nouvelle parcelle.  

### Tester, mesurer et itérer