# README_facturechain_EQUIPE_CM-04_GDJL_Synergy
Il s'agit du dépôt contenant le fichier README du projet de l'équipe CM-04 (GDJL SYNERGY)


# FactureChain

> Plateforme camerounaise de suivi de la consommation électrique des abonnés ENEO — détection automatique d'anomalies, traçabilité des réclamations et vérification d'intégrité des factures par chaîne de hash SHA-256.

---

## Sommaire

1. [Description de la solution](#description-de-la-solution)
2. [Technologies utilisées](#technologies-utilisées)
3. [Architecture globale](#architecture-globale)
4. [Installation & démarrage](#installation--démarrage)
5. [Comptes de démonstration](#comptes-de-démonstration)
6. [Fonctionnalités principales](#fonctionnalités-principales)
7. [API REST — Endpoints](#api-rest--endpoints)
8. [Modèle de données](#modèle-de-données)
9. [Détection d'anomalies](#détection-danomalies)
10. [Ledger & intégrité SHA-256](#ledger--intégrité-sha-256)
11. [Sécurité](#sécurité)
12. [Identité visuelle](#identité-visuelle)
13. [Roadmap](#roadmap)

---

## Description de la solution

FactureChain redonne aux abonnés ENEO Cameroun une visibilité complète sur leur consommation électrique. La plateforme repose sur quatre composants complémentaires :

- **Backend** — API REST Node.js/Express/MongoDB qui centralise les données, détecte les anomalies, gère les réclamations et scelle chaque facture dans une chaîne de hash.
- **App mobile** (React Native + Expo) — accès abonné depuis Android/iOS : tableau de bord, historique des factures, auto-relevé photo, signalement de coupures.
- **Web abonné** (Angular 18) — interface riche avec dashboards, graphiques de consommation, stats publiques par zone.
- **Console admin** (Angular 18) — espace réservé aux agents et administrateurs : ticketing des réclamations, gestion des utilisateurs, modération des coupures, annonces.

Sa philosophie : **être prête le jour où ENEO ouvrira une API**, et fonctionner en mode démonstration jusque-là grâce à un Adapter ENEO `mock ↔ http` switchable sans modifier le reste du code.

---

## Technologies utilisées

### Backend

| Couche | Technologie |
|--------|-------------|
| Runtime | Node.js ≥ 18 |
| Framework | Express |
| Base de données | MongoDB (Mongoose) |
| Authentification | JWT — access token 15 min + refresh token rotatif 7 j |
| Hashage | SHA-256 (chaîne de factures) + bcryptjs (mots de passe) |
| Validation | Joi (`stripUnknown`) |
| Sécurité | Helmet, CORS whitelist, express-rate-limit |
| Logs | Winston + Morgan |
| PDF | PDFKit |

### App mobile

| Couche | Technologie |
|--------|-------------|
| Framework | Expo SDK 51 — React Native 0.74 |
| Langage | TypeScript strict |
| Navigation | React Navigation v6 (native-stack + bottom-tabs) |
| État | Zustand (auth) |
| HTTP | axios + interceptors refresh token |
| Stockage local | AsyncStorage |
| Photos | expo-image-picker |

### Web abonné & Console admin

| Couche | Technologie |
|--------|-------------|
| Framework | Angular 18.2 (standalone components, signals, lazy routes) |
| UI | Angular Material 18.2 + thème custom |
| Langage | TypeScript 5.5 strict |
| HTTP | RxJS 7 + interceptor Bearer + refresh transparent |
| Style | SCSS + variables CSS custom |

---

## Architecture globale

```
FactureChain
├── backend/                    # API REST (Node.js + Express + MongoDB)
│   ├── src/
│   │   ├── config/             # env, database, logger
│   │   ├── models/             # 10 schémas Mongoose
│   │   ├── services/           # logique métier (auth, tariff, ledger, anomaly…)
│   │   ├── controllers/        # handlers HTTP
│   │   ├── routes/             # agrégation Express
│   │   ├── middlewares/        # auth JWT, Joi, erreurs
│   │   ├── validators/         # schémas Joi
│   │   ├── utils/              # helpers (apiError, response, hash, pagination…)
│   │   └── seed/               # données + script de seed
│
├── mobile/                     # App React Native (Expo)
│   └── src/
│       ├── api/                # client axios + endpoints typés
│       ├── components/         # Button, Card, Badge, MeterCard, InvoiceItem…
│       ├── navigation/         # RootNavigator + stacks par tab
│       ├── screens/            # écrans organisés par domaine
│       ├── store/              # Zustand (auth)
│       ├── theme/              # couleurs, typographie, spacing
│       ├── types/              # interfaces miroir du backend
│       └── utils/              # helpers (formatFcfa, formatDate…)
│
├── web-abonne/                 # Web Angular abonné (port 4200)
│   └── src/app/
│       ├── core/               # api, auth (signals), guards, interceptors, models
│       ├── shared/             # badges, layout, pipes (FCFA, kWh, dates)
│       └── pages/              # auth, dashboard, search, meters, invoices,
│                               #  claims, outages, public-stats, profile
│
└── web-admin/                  # Console Angular admin (port 4300)
    └── src/app/
        ├── core/               # admin-api, auth, guards (role), interceptor
        ├── shared/             # badges, layout sombre, pipes
        └── pages/              # auth, dashboard, claims (FSM), users,
                                #  customers, outages, announcements
```

---

## Installation & démarrage

### Prérequis communs

- **Node.js ≥ 18**
- **npm** (ou yarn / pnpm)
- **MongoDB** local (`mongodb://127.0.0.1:27017`) ou Atlas

---

### 1 — Backend

```bash
cd backend

# Installer les dépendances
npm install

# Configurer l'environnement
cp .env.example .env
# Éditez .env : au minimum MONGODB_URI et les secrets JWT

# Charger les données de démo
npm run seed:reset

# Lancer en mode développement (rechargement à chaud)
npm run dev
```

L'API est disponible sur `http://localhost:4000/api/v1`.

Vérification rapide :
```bash
curl http://localhost:4000/api/v1/health
```

---

### 2 — Web abonné (Angular)

```bash
cd web-abonne
npm install
npm start
# → http://localhost:4200
```

Build de production :
```bash
npm run build
# Bundle généré dans dist/facture-chain-web/browser/
```

Par défaut l'app pointe vers `http://localhost:4000/api/v1` (dev) ou `/api/v1` (prod, derrière reverse proxy). Pour changer : modifiez `apiBaseUrl` dans `src/environments/`.

---

### 3 — Console admin (Angular)

```bash
cd web-admin
npm install
npm start
# → http://localhost:4300
```

---

### 4 — App mobile (Expo)

```bash
cd mobile
npm install
npx expo start
```

Une fois Expo lancé :

- **Téléphone réel** : scannez le QR code avec **Expo Go**
- **Émulateur Android** : appuyez sur `a`
- **Simulateur iOS** : appuyez sur `i` (macOS uniquement)

**Configuration de l'URL backend :**

| Cible | URL par défaut |
|-------|----------------|
| iOS Simulator | `http://localhost:4000` |
| Émulateur Android | `http://10.0.2.2:4000` |
| Téléphone réel | À surcharger via la variable d'environnement |

```bash
# Sur téléphone réel — remplacez l'IP par celle de votre machine hôte
EXPO_PUBLIC_API_URL=http://192.168.1.42:4000 npx expo start
```

---

## Comptes de démonstration

Mot de passe commun pour tous les comptes abonnés : **`Demo@2024`**

| Rôle | Email | Mot de passe | Profil |
|------|-------|-------------|--------|
| Admin | `admin@facturechain.cm` | `Admin@2024` | Console admin complète |
| Agent | `agent@facturechain.cm` | `Agent@2024` | Console agent (tickets, modération) |
| Abonné | `marie.atangana@example.cm` | `Demo@2024` | Yaoundé · Bastos · 1 compteur · anomalies pré-injectées ✅ |
| Abonné | `patrick.mbarga@example.cm` | `Demo@2024` | Douala · 2 compteurs (Akwa + Bonapriso) · anomalies ✅ |
| Abonné | `sandra.eboa@example.cm` | `Demo@2024` | Bafoussam |
| Abonné | `francis.ngono@example.cm` | `Demo@2024` | Yaoundé · Mendong |
| Abonné | `rosine.fokou@example.cm` | `Demo@2024` | Bamenda |
| Abonné | `boris.ekane@example.cm` | `Demo@2024` | Douala · Bonanjo (commercial) |

> ⚠️ Ces comptes sont en lecture/écriture sur la base de démo. Évitez d'y enregistrer des données sensibles.

---

## Fonctionnalités principales

### 🔍 Recherche de facture instantanée
Recherchez n'importe quelle facture par son numéro (ex : `ENEO-2024-08-YDE-000123`). Affichage immédiat du recalcul FactureChain (écart vs ENEO) et des anomalies détectées — accessible sans rattachement de compteur.

### ⚡ Suivi des compteurs
Pour chaque compteur : graphique de consommation 12 mois, liste des factures, anomalies en cours, KPIs (index actuel, conso 12 mois, moyenne mensuelle), historique des relevés.

### 🔐 Vérification d'intégrité SHA-256
Chaque facture porte une empreinte SHA-256 chaînée à la précédente (style ledger). Bouton one-click pour confirmer qu'une facture n'a pas été altérée. Le hash est imprimé en pied du PDF.

### 📋 Réclamations end-to-end
Création par type (contestation / défaut compteur / erreur relevé / coupure / raccordement / autre), suivi timeline avec historique de statuts en machine d'état (8 étapes), conversation bidirectionnelle abonné ↔ agent.

### 🌍 Coupures communautaires
Signalement de coupures avec géolocalisation (région / ville / quartier), confirmation par d'autres utilisateurs (effet réseau, auto-promotion à 3 confirmations), suivi des résolutions.

### 📊 Stats publiques
Indicateurs anonymisés sur les anomalies de facturation et les coupures par zone (30 j). Pression collective sur la qualité du service.

### 📸 Auto-relevé photo (mobile)
Soumission d'un relevé avec photo + index + notes. Permet la détection de divergences avec les relevés ENEO.

### 🛠️ Console d'administration
Tableau de bord KPI temps réel, console ticketing réclamations (FSM contrôlé), gestion des utilisateurs (suspension/réactivation), annuaire clients ENEO, modération des signalements de coupures, publication d'annonces.

---

## API REST — Endpoints

Préfixe : `/api/v1`

### Authentification

| Méthode | Route | Description |
|---------|-------|-------------|
| POST | `/auth/register` | Inscription (avec ou sans `clientId` ENEO) |
| POST | `/auth/login` | Login → `accessToken` + `refreshToken` |
| POST | `/auth/refresh` | Rafraîchit l'access token |
| POST | `/auth/logout` | Révoque le refresh token courant |
| GET | `/auth/me` | Profil + fiche client liée |

### Abonné

| Méthode | Route | Description |
|---------|-------|-------------|
| GET | `/me/customer` | Ma fiche client |
| GET | `/me/meters` | Mes compteurs |
| GET | `/meters/:meterId` | Détail d'un compteur |
| GET | `/meters/:meterId/invoices` | Historique des factures (paginé) |
| GET | `/meters/:meterId/index-readings` | 60 derniers relevés |
| POST | `/index-readings` | Soumettre un auto-relevé |
| GET | `/meters/:meterId/anomalies` | Anomalies détectées |
| GET | `/meters/:meterId/consumption-stats` | Série mensuelle 12 mois |
| GET | `/meters/:meterId/verify-chain` | Vérification de la chaîne de hash |

### Factures

| Méthode | Route | Description |
|---------|-------|-------------|
| GET | `/invoices/search?invoiceNumber=…` | Recherche par numéro |
| GET | `/invoices/:invoiceId` | Détail facture + anomalies |
| GET | `/invoices/:invoiceId/verify` | Vérification empreinte SHA-256 |
| GET | `/invoices/:invoiceId/pdf` | Téléchargement PDF |

### Réclamations

| Méthode | Route | Description |
|---------|-------|-------------|
| POST | `/claims` | Soumettre une réclamation |
| GET | `/claims/mine` | Mes réclamations |
| GET | `/claims/:claimId` | Détail (messages + historique) |
| POST | `/claims/:claimId/messages` | Ajouter un message |

### Coupures

| Méthode | Route | Description |
|---------|-------|-------------|
| GET | `/outages` | Liste publique |
| POST | `/outages` | Signaler une coupure |
| POST | `/outages/:id/confirm` | Confirmer (+1 ; auto-promotion à 3) |
| POST | `/outages/:id/resolve` | Marquer résolu (agent/admin) |

### Stats publiques

| Méthode | Route | Description |
|---------|-------|-------------|
| GET | `/public/stats/anomalies-by-zone` | Anomalies par ville (30 j) |
| GET | `/public/stats/outages-by-zone` | Coupures par ville (30 j) |

### Admin / Agent

| Méthode | Route | Rôle requis | Description |
|---------|-------|-------------|-------------|
| GET | `/admin/dashboard` | agent/admin | KPIs temps réel |
| GET | `/admin/users` | admin | Liste utilisateurs |
| PATCH | `/admin/users/:userId/status` | admin | Suspendre/réactiver un compte |
| GET | `/admin/customers` | agent/admin | Recherche clients |
| GET | `/admin/claims` | agent/admin | Console réclamations |
| PATCH | `/admin/claims/:claimId/status` | agent/admin | Changer le statut (FSM) |
| PATCH | `/admin/claims/:claimId/assign` | admin | Assigner à un agent |
| GET | `/admin/announcements` | agent/admin | Liste des annonces |
| POST | `/admin/announcements` | admin | Créer une annonce |

---

## Modèle de données

Dix collections MongoDB :

| Collection | Rôle |
|------------|------|
| **User** | Compte de connexion (email, password hashé, rôle) |
| **Customer** | Fiche client ENEO (clientId, adresse, type) |
| **Meter** | Compteur (numéro unique, catégorie tarifaire, dernier index) |
| **Invoice** | Facture scellée (hash, previousHash, breakdown tarifaire, écart facturé/recalculé) |
| **IndexReading** | Relevés (ENEO / auto-relevé / seed) |
| **Anomaly** | Anomalies détectées (6 types, 3 sévérités) |
| **Claim** | Réclamation tracée (8 statuts, FSM contrôlé, fil de messages, lien ENEO) |
| **PowerOutage** | Signalements communautaires de coupures |
| **Announcement** | Annonces ENEO/admin (ciblage par zone) |
| **AuditLog** | Journal d'audit (prêt à brancher) |

---

## Détection d'anomalies

Lancée automatiquement à chaque création ou import de facture. Six types détectés :

| Type | Critère |
|------|---------|
| `NEGATIVE_INCREMENT` | Index courant < index précédent |
| `CONSUMPTION_SPIKE` | Z-score ≥ 2 sur l'historique 12 mois (high si ≥ 3) |
| `YOY_DEVIATION` | Hausse ≥ +50 % vs même mois N-1 |
| `IMPOSSIBLE_CONSUMPTION` | > 3 000 kWh/mois sur un compteur résidentiel |
| `AMOUNT_MISMATCH` | Écart ≥ 5 % entre montant facturé et recalcul (high si ≥ 20 %) |
| `MISSING_READING` | Période sans relevé |

Les seuils sont centralisés dans `src/services/anomaly.service.js`.

Les anomalies des 30 derniers jours sont agrégées par zone dans `/public/stats/anomalies-by-zone`, exposant une pression collective sur la qualité du service.

---

## Ledger & intégrité SHA-256

Chaque facture stocke `hash` et `previousHash`. Le hash est calculé en SHA-256 sur les champs canoniques (numéro, compteur, période, index avant/après, consommation, montant, dates) **+ le `previousHash`** de la dernière facture du même compteur. Toute modification ultérieure casse la chaîne.

Endpoints de vérification :
- `GET /invoices/:id/verify` — recalcule le hash et le compare au stocké.
- `GET /meters/:id/verify-chain` — parcourt la chronologie complète et retourne `{ valid, brokenAt? }`.

**Mode `LEDGER_MODE=blockchain`** — le service `ledger.service.js` contient le crochet pour publier le hash sur une blockchain (Polygon, Hyperledger Fabric, contrat personnalisé). Ce stub log uniquement aujourd'hui ; il suffit de le brancher sans toucher au reste du code.

### Machine d'état des réclamations

```
submitted → received | rejected
received → investigating | rejected
investigating → transmitted_to_eneo | resolved | rejected
transmitted_to_eneo → awaiting_response | resolved
awaiting_response → resolved | rejected
resolved → closed
rejected → closed
closed → ∅
```

Les transitions invalides sont rejetées par le backend ; l'UI ne propose que les transitions autorisées.

---

## Sécurité

- **Helmet** + **CORS whitelist** + **rate limiting** (300 req / 15 min par IP)
- **JWT** : access token 15 min + refresh token rotatif 7 j (5 tokens max par compte)
- **bcrypt** sur les mots de passe (10 rounds)
- **Joi** systématique sur les payloads sensibles (`stripUnknown`)
- Les abonnés ne voient que leurs propres factures, compteurs et réclamations
- Tokens stockés en `AsyncStorage` (mobile) et `localStorage` (web)
- Interceptors HTTP pour injection automatique du Bearer et refresh transparent
- Guards de rôle : la console admin refuse les connexions avec un compte abonné

À ajouter pour la production :
- 2FA par OTP SMS (pertinent au Cameroun)
- Audit log automatique sur les actions sensibles
- HTTPS + cookies `secure` pour le refresh token
- Renforcement CSP/HSTS via Helmet

---

## Identité visuelle

| Rôle | Couleur |
|------|---------|
| Primaire | `#0B5FFF` (énergie, confiance) |
| Secondaire | `#0FB57A` (succès, validations) |
| Warning | `#F59E0B` |
| Danger | `#E53935` |
| Fond | `#F5F7FB` |

- **Typographie** : Inter (Google Fonts)
- **Icônes** : Material Symbols Outlined
- **Radius** : 6 / 10 / 14 / 20 px selon les composants
- **Ombres** : 2 niveaux (sm, md)

---

## Roadmap

| Phase | Statut | Description |
|-------|--------|-------------|
| Phase 1 — Backend | ✅ Livré | API REST complète |
| Phase 2 — Mobile | ✅ Livré | App React Native (abonné) |
| Phase 3 — Web abonné | ✅ Livré | Angular dashboards |
| Phase 4 — Console admin | ✅ Livré | Ticketing + modération |
| Phase 5 — Documents | À venir | Présentation Word + PDF |
| Phase 6 — Blockchain + ENEO API | À venir | Ancrage hash réel + bascule adapter http |

Fonctionnalités post-MVP prévues :
- Notifications push (nouvelle facture, anomalie, mise à jour réclamation)
- Carte interactive des coupures (react-native-maps)
- Mode hors ligne (offline-first, react-query + cache persistant)
- Authentification biométrique (FaceID / empreinte)
- OTP SMS pour la création de compte
- Multi-langue (français / anglais / langues locales)

---

© FactureChain — Suivi de votre consommation électrique · Cameroun
