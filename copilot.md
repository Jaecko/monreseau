Salut Copilot. Nous allons construire ensemble une application SaaS complexe. Voici le cahier des charges complet (notre "master plan"). Je te demanderai de générer du code, fichier par fichier, en te basant sur cette architecture.

### 1. Objectif du Projet : "MonReseau"

-   **Produit :** Une application (Web + Mobile) pour la création de réseaux sociaux privés pour les réseaux d'entrepreneurs.
-   **Business Model :** SaaS avec 3 niveaux de plans (Standard, Pro, Enterprise).
-   **Multi-Tenancy :** L'application est multi-tenant. La donnée est cloisonnée par `network_id`.
-   **Multi-Profil :** Un `User` peut appartenir à plusieurs `Network`. L'interface (web et mobile) doit permettre de switcher entre ces réseaux.

### 2. Architecture Technique (Stack)

1.  **Backend (API-Only) :** **Laravel 11**.
    -   **Auth :** Laravel Sanctum (pour l'authentification par token).
    -   **Paiement :** Laravel Cashier (Stripe) pour la gestion des abonnements.
    -   **WebSockets (V1) :** **Laravel Reverb** (pour les notifications temps réel).
    -   **Base de données :** MySQL 8.
    -   **Cache & Queues :** Redis (pour les jobs, emails, notifications).

2.  **Frontend (Single Codebase) :** **Vue.js 3** (Composition API).
    -   **State Management :** Pinia.
    -   **Routing :** Vue Router.
    -   **API Client :** Axios (avec intercepteurs pour le token et le `X-Network-ID`).
    -   **WebSocket Client :** **Laravel Echo** (pour se connecter à Reverb).

3.  **Mobile (iOS/Android) :** **Capacitor**.
    -   Nous "enveloppons" l'application Vue.js avec Capacitor pour la publier sur les stores.
    -   Nous utiliserons le plugin `capacitor-push-notifications` (V2) et `capacitor-browser` (V1, pour les paiements).

### 3. Infrastructure de Développement (Docker)

Nous utilisons `docker-compose.yml` avec les services suivants :
-   `nginx` : Reverse proxy. Il sert les fichiers statiques de Vue et proxy les requêtes `/api/` vers le backend.
-   `backend` : PHP 8.2-FPM (notre app Laravel).
-   `db` : MySQL 8.
-   `redis` : Cache et Queues.
-   `mailhog` : Faux serveur SMTP pour le développement.
-   `reverb` : Le serveur WebSocket Laravel Reverb (pour les notifications V1).

### 4. Schéma de Données & Modèles Clés

-   `users` : Table standard.
-   `plans` : Les forfaits d'abonnement.
    -   `name` (string)
    -   `price_monthly` (decimal)
    -   `user_limit` (integer)
    -   `stripe_plan_id` (string)
    -   `features` (json) : C'est notre système de "Feature Flags". Ex: `{"gamification": true, "dynamic_branding": true, "white_label_app": false}`.
-   `networks` (Les "Tenants") :
    -   `name` (string)
    -   `owner_id` (FK vers `users`)
    -   `plan_id` (FK vers `plans`)
    -   `branding` (json, nullable) : Stocke les assets de branding. **C'est la clé de la V2**.
        -   Ex: `{"primary_color": "#A11B2F", "logo_url": "...", "app_name": "Réseau 1 App", "app_id": "com.reseau1"}`.
    -   **Doit utiliser le Trait `Billable` de Cashier**. C'est le `Network` qui est abonné.
-   `network_user` (Table Pivot) :
    -   `network_id`, `user_id`, `role` ('owner', 'admin', 'member').
-   `posts` : Doit avoir `user_id` ET `network_id`.
-   `network_transfers` : Pour le transfert de propriété sécurisé (`network_id`, `from_user_id`, `to_user_id`, `status`, `token`).
-   `notifications` : Table standard de Laravel Notifications.

### 5. Logique Métier & Fonctionnalités Clés

1.  **Flux d'Authentification (Login) :**
    -   L'API `POST /api/login` renvoie le `user` et `user.networks[]`.
    -   **Frontend :** Si `networks.length == 1`, on sélectionne ce réseau automatiquement. Si `networks.length > 1`, on redirige l'utilisateur vers la page `/select-network`.

2.  **Cloisonnement des Données (Multi-Tenancy) :**
    -   Une fois un réseau sélectionné, le frontend (Pinia) stocke le `selectedNetworkId` et ses détails (y compris les `features` de son plan et son `branding`).
    -   L'intercepteur Axios envoie le header `X-Network-ID` à CHAQUE requête API.
    -   **Backend :** Nous créons un **Middleware** `EnsureUserBelongsToNetwork` qui lit ce header et vérifie l'appartenance.

3.  **Flux d'Abonnement (SaaS) :**
    -   L'utilisateur s'inscrit (crée un `User`).
    -   Il arrive sur une page "Créer un réseau".
    -   Il choisit un `Plan`, entre le nom de son réseau et ses infos de paiement (Stripe.js).
    -   L'API `POST /api/subscribe` crée le `Network`, abonne le `Network` au plan via Cashier, et lie le `User` comme 'owner' dans la table pivot.

4.  **Logique de Branding Stratifié (V2) :**
    -   **Plan Standard :** `features.dynamic_branding: false`. L'application affiche le thème "MonReseau" par défaut.
    -   **Plan Pro :** `features.dynamic_branding: true`.
        -   Quand l'utilisateur sélectionne ce réseau dans l'application principale, le frontend (Vue) lit les données de `network.branding`.
        -   Il utilise un "Theme Manager" pour mettre à jour dynamiquement des **CSS Custom Properties** (ex: `:root { --primary-color: #... }`) et le logo dans le header.
        -   Quand l'utilisateur change pour un réseau "Standard", le thème revient à la normale.
    -   **Plan Enterprise :** `features.white_label_app: true`.
        -   L'owner obtient tout ce qui est dans le "Pro", *PLUS* :
        -   Nous utilisons un **Script d'Usine** (CI/CD) pour lire ces *mêmes* données de `branding` afin de générer une application mobile (`.apk`/`.ipa`) dédiée (changement du `capacitor.config.json`, des icônes, etc.).

5.  **Notifications (V1) :**
    -   Nous utilisons Laravel Notifications avec les canaux `database` (pour la cloche 🔔), `mail` (emails en file d'attente), et **`broadcast`** (pour le temps réel).
    -   Le backend utilise **Reverb** pour pousser les notifications.
    -   Le frontend (Vue) utilise **Laravel Echo** pour les écouter et mettre à jour le store Pinia instantanément.

6.  **Transfert de Propriété (Sécurisé - V1) :**
    -   Flux en 6 étapes : Initier (Owner A) -> Générer `token` (Backend) -> Email (User B) -> Accepter + Entrer Paiement (User B) -> Finaliser (Backend, `swap()` de Cashier + update `owner_id`).

7.  **Paiements Mobiles (Capacitor - V1) :**
    -   **RÈGLE IMPORTANTE :** Nous n'utilisons **PAS** les achats In-App (IAP).
    -   Dans l'application mobile, les boutons "S'abonner" ou "Gérer l'abonnement" ouvrent le site web (`app.monreseau.be`) dans un **navigateur In-App** (Capacitor Browser plugin).

### 6. DevOps (CI/CD - V1)

-   **CI (Intégration Continue) :** Nous utilisons **GitHub Actions**.
    -   À chaque `git push`, un workflow doit automatiquement :
        1.  Installer les dépendances (Composer & NPM).
        2.  Lancer les tests (Pest/PHPUnit) : `php artisan test`.
        3.  Linter le code.
-   **CD (Déploiement Continu) :**
    -   Nous utiliserons un service comme **Ploi.io** ou **Render.com**.
    -   Quand la CI est au vert ✅ sur la branche `main`, un webhook déclenche le déploiement.
    -   Le déploiement doit être **zéro-downtime**.

Ceci est notre plan directeur. Je vais maintenant commencer à créer la structure des fichiers et te demander de m'aider à les remplir. Es-tu prêt ?
