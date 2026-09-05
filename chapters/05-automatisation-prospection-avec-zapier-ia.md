## 1️⃣ Capture des leads depuis Facebook Lead Ads  

### 1.1 Créer le formulaire de capture  

- Dans le **Business Manager** de Facebook, créez une campagne « Lead Generation ».  
- Ajoutez les champs indispensables : prénom, nom, email, numéro de téléphone, ville.  
- Activez l’option **« Instant Form »** et choisissez **« Webhook »** comme destination des données.  

### 1.2 Connecter Facebook à Zapier  

1. Ouvrez votre compte Zapier (ou Make) et cliquez sur **« Make a Zap »**.  
2. **Trigger** : *Facebook Lead Ads – New Lead*.  
3. Sélectionnez la page et le formulaire créés précédemment.  
4. Testez la connexion : Zapier récupère le dernier lead soumis.  

> **Astuce Empire du Web** : si vous avez plusieurs pages (ex. : une page par pays – Nigéria, Kenya, Côte d’Ivoire), créez un seul Zap avec un **Filter** qui ne poursuit que si le champ *Pays* correspond à la zone ciblée.  

---

## 2️⃣ Enrichissement des contacts avec Clearbit  

### 2.1 Pourquoi enrichir ?  

Un lead contenant uniquement un email est souvent incomplet : vous ne connaissez pas la taille de l’entreprise, le secteur d’activité ou le revenu annuel. Clearbit **enrichit** automatiquement ces informations, ce qui rend vos séquences d’email plus pertinentes.  

### 2.2 Configurer l’action Clearbit  

1. Dans le même Zap, ajoutez une **Action** : *Clearbit – Enrich Person*.  
2. Mappez le champ *email* du lead Facebook vers le champ *Email* de Clearbit.  
3. Activez les champs d’enrichissement utiles : `companyName`, `companyDomain`, `companyIndustry`, `companySize`, `location`.  

> **Remarque** : Clearbit propose un plan gratuit limité à 100 enrichissements/mois, suffisant pour un petit business.  

### 2.3 Gestion des erreurs d’enrichissement  

- **Filter** : si le champ `companyDomain` est vide, passez le lead dans une branche « Enrichissement manuel ».  
- **Path** (Zapier) : créez deux chemins – *Enrichi* et *Non enrichi*. Le premier continue le workflow, le second envoie une notification Slack à l’équipe de vente pour traitement manuel.  

---

## 3️⃣ Rédaction d’e‑mails personnalisés avec GPT‑3  

### 3.1 Préparer le prompt  

Le secret d’un texte généré de qualité réside dans le **prompt**. Un bon prompt inclut :  

- Le ton (professionnel, chaleureux, africain).  
- Le contexte (secteur, taille d’entreprise).  
- L’objectif (demande de RDV, présentation de produit).  

Exemple de prompt :  

```text
Tu es un commercial spécialisé dans les solutions SaaS pour les PME africaines. Rédige un e‑mail de 150 mots, en français, à destination d’un dirigeant d’une entreprise de 30 employés dans le secteur de la logistique à Nairobi. Le message doit présenter notre plateforme de suivi de livraisons, mentionner le bénéfice d’une réduction de 20 % des coûts de transport, et proposer un appel de 15 minutes la semaine prochaine.
```

### 3.2 Créer l’action OpenAI dans Zapier  

1. **Action** : *OpenAI – Create Completion*.  
2. Sélectionnez le modèle **gpt‑3.5‑turbo** (ou **gpt‑4** si disponible).  
3. Dans le champ *Prompt*, insérez le texte ci‑dessus en remplaçant les variables par les valeurs du lead :  

```json
{
  "prompt": "Tu es un commercial spécialisé dans les solutions SaaS pour les PME africaines. Rédige un e‑mail de 150 mots, en français, à destination de {{first_name}} {{last_name}} qui dirige {{company_name}} ({{company_industry}}) à {{city}}. ..."
}
```

4. Définissez **temperature** à 0.7 pour garder un ton créatif mais contrôlé.  

### 3.3 Stocker le texte généré  

Ajoutez une **Action** : *Google Sheets – Create Spreadsheet Row* (ou *Airtable*).  
- Colonnes : `Email`, `Nom`, `Entreprise`, `Objet`, `Corps`, `Statut` (initialisé à « Prêt à envoyer »).  
- Le corps de l’e‑mail provient du champ **Choices[0].text** renvoyé par OpenAI.  

> **Tip** : Conservez le **prompt complet** dans une colonne « Prompt » afin de pouvoir le ré‑utiliser pour l’A/B testing.  

---

## 4️⃣ Envoi automatisé via Gmail ou Outlook  

### 4.1 Choisir le canal d’envoi  

- **Gmail** : idéal pour les petites équipes, limite de 2 000 mails/jour.  
- **Outlook/Office 365** : plus adapté aux entreprises disposant d’un domaine professionnel.  

### 4.2 Configurer l’action d’envoi  

1. **Action** : *Gmail – Send Email* (ou *Microsoft Outlook – Send Email*).  
2. Mappez :  
   - `To` → `email` du lead.  
   - `Subject` → `"{{company_name}} – Optimisez vos livraisons en 2024"` (exemple).  
   - `Body` → le texte généré stocké dans Google Sheets.  
3. Ajoutez les **headers** d’authentification DKIM/SPF si vous avez votre propre domaine (voir chapitre 6 pour la configuration des DNS).  

### 4.3 Gestion des rebonds et désinscriptions  

- **Filter** : si le champ `Statut` passe à « Bounced », routez le lead vers une liste « À nettoyer » dans Mailchimp.  
- **Action** : *Zapier – Update Spreadsheet Row* → change `Statut` à « Envoyé ».  

---

## 5️⃣ Programmation de séquences de suivi  

### 5.1 Créer la logique de timing  

Zapier ne propose pas de **delays** supérieurs à 30 jours, mais vous pouvez combiner **Delay Until** et **Schedule** pour créer des séquences :  

| Étape | Délai | Action |
|------|------|--------|
| 1 | 0 j | Envoi du premier e‑mail (déjà configuré). |
| 2 | +3 j | Envoi d’un rappel « Avez‑vous reçu mon précédent message ? ». |
| 3 | +7 j | Envoi d’une offre de contenu (e‑book sur la logistique). |
| 4 | +14 j | Notification à l’équipe de vente pour appel téléphonique. |

### 5.2 Implémenter le rappel automatique  

1. Ajoutez un **Zap** séparé : *Schedule – Every Day*.  
2. **Find Row** dans Google Sheets où `Statut = "Envoyé"` et `Date d’envoi = aujourd’hui - 3 jours`.  
3. **Action** : *Gmail – Send Email* avec un nouveau prompt :  

```text
Rédige un e‑mail de suivi bref, rappelant le précédent message, en insistant sur le bénéfice de notre solution pour {{company_name}}.
```  

4. Mettez à jour la colonne `Statut` à « Rappel 1 ».  

### 5.3 Passage à la vente  

Lorsque le lead atteint le **4ᵉ** rappel, créez un **Path** :  

- **Si** `Réponse = "Oui"` → créez une tâche dans **HubSpot** ou **Pipedrive** (action *Create Deal*).  
- **Sinon** → marquez le lead comme « Non intéressé » et archivez.  

---

## 6️⃣ Tableau de bord de suivi (Google Data Studio)  

### 6.1 Connecter les données  

- Dans **Google Data Studio**, ajoutez votre Google Sheet comme source.  
- Créez des métriques : `Leads capturés`, `Leads enrichis`, `Emails envoyés`, `Taux d’ouverture`, `Taux de réponse`.  

### 6.2 Visualiser la performance par pays  

Utilisez un **graphique à barres** avec le champ `Country`. Vous pourrez ainsi identifier les zones où le taux de conversion est le plus élevé (ex. : Ghana : 12 %, Kenya : 8 %).  

> **Insight** : si un pays montre un taux d’ouverture faible, testez un objet d’e‑mail en **langue locale** (swahili, haoussa, etc.).  

---

## 7️⃣ Bonnes pratiques et pièges à éviter  

### 7.1 Respect de la législation locale  

- **Nigeria** : le **Nigeria Data Protection Regulation (NDPR)** exige le consentement explicite avant l’envoi d’e‑mails marketing.  
- **Kenya** : le **Data Protection Act** impose un délai de 30 jours pour la suppression des données sur demande.  

Intégrez une **case à cocher consentement** dans le formulaire Facebook et stockez la valeur `consent = true` dans votre sheet.  

### 7.2 Gestion du volume d’appels d’API  

- **Clearbit** et **OpenAI** facturent à la requête. Utilisez un **Filter** pour ne pas appeler OpenAI si le lead provient d’une entreprise déjà très connue (ex. : `companyDomain` = "google.com").  
- Mettez en place un **Cache** : créez une table “Enrichissements” où chaque email déjà enrichi est stocké. Avant chaque appel, cherchez d’abord dans cette table.  

### 7.3 Optimisation du taux d’ouverture  

- **Objet dynamique** : utilisez le nom de l’entreprise ou du dirigeant (`{{company_name}} – {{first_name}}`).  
- **Heure d’envoi** : testez les créneaux locaux (ex. : 9 h–11 h, 16 h–18 h). Zapier permet de programmer l’envoi avec *Delay Until* en fonction du fuseau horaire (`{{city}}`).  

---

## 8️⃣ Étude de cas : *Kofi*, fondateur d’une startup logistique au Ghana  

| Étape | Action | Résultat |
|------|--------|----------|
| Capture | Formulaire Facebook → Zapier | 250 leads en 2 semaines |
| Enrichissement | Clearbit (plan gratuit) | 180 leads enrichis, 70 % d’entre eux avec `companySize > 20` |
| Rédaction | GPT‑3 (temperature = 0.6) | Taux d’ouverture 34 % vs 22 % moyen du secteur |
| Envoi | Gmail (domain `kofi-logistics.com`) | 45 % de réponses, 12 RDV qualifiés |
| Suivi | Séquence 3 rappels + Slack → équipe vente | 6 contrats signés, revenu supplémentaire de 12 000 USD/mois |

Ce cas montre que **l’automatisation** ne remplace pas la touche humaine : chaque RDV a été conduit par un commercial qui a personnalisé la proposition finale.  

---

## 9️⃣ Points clés à retenir  

- **Intégrer** Facebook Lead Ads → Zapier → Clearbit → OpenAI → Gmail crée un **pipeline complet** sans écrire de code.  
- **Enrichir** les leads avant la rédaction d’e‑mail augmente le taux d’ouverture de plus de 10 %.  
- Un **prompt bien structuré** (ton, contexte, objectif) est la clé pour obtenir des e‑mails convaincants avec GPT‑3.  
- **Segmenter** les séquences de suivi (3 jours, 7 jours, 14 jours) et automatiser les notifications à l’équipe de vente maximise les chances de conversion.  
- **Respecter** les réglementations locales (NDPR, Data Protection Act) en collectant le consentement dès le formulaire.  
- **Surveiller** les indicateurs (ouverture, réponse, conversion) dans un tableau de bord Data Studio pour ajuster les objets, le timing et le ton en fonction des marchés africains ciblés.  

En appliquant ces étapes, vous transformerez vos listes de prospects en un **flux de ventes automatisé**, libérant du temps pour vous concentrer sur la création de valeur et l’expansion de votre entreprise sur le continent. 🚀