Salut Copilot. Nous allons construire ensemble une application SaaS complexe. Voici le cahier des charges complet (notre "master plan"). Je te demanderai de générer du code, fichier par fichier, en te basant sur cette architecture.

### 1. Objectif du Projet : "MonReseau" (Nom Provisoire)

-   **Produit :** Une application (Web + Mobile) pour la création de réseaux sociaux privés pour les réseaux d'entrepreneurs.
-   **Business Model :** SaaS avec 3 niveaux de plans (Standard, Pro, Enterprise).
-   **Différenciateur Clé (V2) :** Un **système de validation croisée** pour les opportunités d'affaires. L'objectif est de prouver le ROI financier du réseau grâce à la reconnaissance sociale et à la confirmation mutuelle, et non à l'auto-déclaration.
-   **Multi-Tenancy :** L'application est multi-tenant. La donnée est cloisonnée par `network_id`.
-   **Multi-Profil :** Un `User` peut appartenir à plusieurs `Network`.

### 2. Architecture Technique (Stack)

1.  **Backend (API-Only) :** **Laravel 11**.
    -   **Auth :** Laravel Sanctum.
    -   **Paiement :** Laravel Cashier (Stripe).
    -   **WebSockets (V1) :** **Laravel Reverb** (pour les notifications temps réel).
    -   **Base de données :** MySQL 8.
    -   **Cache & Queues :** Redis.
2.  **Frontend (Single Codebase) :** **Vue.js 3** (Composition API).
    -   **State Management :** Pinia.
    -   **Routing :** Vue Router.
    -   **API Client :** Axios.
    -   **WebSocket Client :** **Laravel Echo**.
3.  **Mobile (iOS/Android) :** **Capacitor**.

### 3. Infrastructure de Développement (Docker)

Nous utilisons `docker-compose.yml` avec 6 services :
-   `nginx` (Reverse proxy), `backend` (PHP-FPM), `db` (MySQL), `redis`, `mailhog`, `reverb` (WebSocket).

### 4. Schéma de Données & Modèles Clés

-   `users` : Table standard.
-   `plans` : Les forfaits d'abonnement.
    -   `name` (string), `price_monthly` (decimal), `user_limit` (integer), `stripe_plan_id` (string).
    -   `features` (json) : C'est notre système de "Feature Flags".
        -   Ex: `{"gamification": true, "dynamic_branding": true, "white_label_app": false, "opportunity_tracker": true}`.
-   `networks` (Les "Tenants") :
    -   `name` (string), `owner_id` (FK `users`), `plan_id` (FK `plans`).
    -   `branding` (json, nullable) : Stocke les assets de branding (couleurs, logo, app_id).
    -   **Doit utiliser le Trait `Billable` de Cashier**. C'est le `Network` qui est abonné.
-   `network_user` (Table Pivot) :
    -   `network_id`, `user_id`, `role` ('owner', 'admin', 'member').
-   `posts` : Doit avoir `user_id` ET `network_id`.
-   `network_transfers` : Pour le transfert de propriété sécurisé (`network_id`, `from_user_id`, `to_user_id`, `status`, `token`).
-   `notifications` : Table standard de Laravel Notifications.
-   `opportunities` (Table V2 - Clé du projet) :
    -   `network_id` (FK `networks`)
    -   `from_user_id` (FK `users`, le "Donneur")
    -   `to_user_id` (FK `users`, le "Receveur")
    -   `contact_name` (string)
    -   `description` (text)
    -   `status` (enum: 'Sent', 'In Progress', **'Pending Confirmation'**, **'Confirmed'**, 'Disputed', 'Lost', 'Archived') - Default: 'Sent'.
    -   `value` (decimal, 10, 2, nullable) : Montant du deal si `Confirmed`.

### 5. Logique Métier & Fonctionnalités Clés

1.  **Flux d'Authentification (Login) :** (Standard)
    -   L'API `POST /api/login` renvoie le `user` et `user.networks[]`.
    -   Le Frontend gère la redirection (`SelectNetwork.vue` si > 1 réseau).

2.  **Cloisonnement des Données (Multi-Tenancy) :** (Standard)
    -   Le Frontend envoie le header `X-Network-ID`.
    -   Le Backend vérifie l'appartenance avec le **Middleware** `EnsureUserBelongsToNetwork`.

3.  **Flux d'Abonnement (SaaS) :** (Standard)
    -   `POST /api/subscribe` crée le `Network`, abonne le `Network` à Cashier, et lie le `User` comme 'owner'.

4.  **Module "Opportunity Tracker" (V2) - Logique de Validation Croisée :**
    -   Cette fonctionnalité est protégée par le `feature_flag` "opportunity_tracker".
    -   `User A` (Donneur) crée une `Opportunity` pour `User B` (Receveur). Statut = `Sent`. `User B` est notifié (via Reverb).
    -   `User B` peut changer le statut. S'il choisit `Won`, le statut passe à `Pending Confirmation` et il **doit** saisir une `value`.
    -   Le système notifie (via Reverb) `User A` (le Donneur).
    -   `User A` voit l'opportunité et doit "Confirmer" ou "Discuter" (`Disputed`).
    -   **Si `User A` confirme :**
        1.  Le statut passe à `Confirmed`.
        2.  Le backend déclenche un `Event` (ex: `OpportunityConfirmed`).
        3.  Un `Listener` capture cet event et :
            a. Attribue les points de gamification à A et B.
            b. **Auto-publie un `Post`** (sans montant) sur le fil d'actualité pour la reconnaissance sociale (ex: "[User B] a gagné une affaire grâce à [User A]").
    -   **Dashboard Admin :** L'Admin du réseau a accès à une route API qui renvoie le `SUM(value)` et le `COUNT()` des opportunités `Confirmed` pour son `network_id`.
    -   **Confidentialité :** Le `value` reste toujours privé (visible seulement par A, B, et les Admins du réseau).

5.  **Logique de Branding Stratifié (V2) :** (Standard)
    -   **Plan Pro :** `dynamic_branding: true`. Le Frontend (Vue) applique le thème (CSS) en fonction du `network.branding` sélectionné.
    -   **Plan Enterprise :** `white_label_app: true`. Un "Script d'Usine" (CI/CD) compile une app mobile dédiée en utilisant ces mêmes données.

6.  **Notifications (V1) :**
    -   Canaux `database`, `mail` (via Queues), et **`broadcast`** (via **Reverb**).
    -   Le Frontend (Vue) utilise **Laravel Echo** pour l'écoute en temps réel.

7.  **Transfert de Propriété (Sécurisé - V1) :** (Standard)
    -   Flux en 6 étapes : Initier -> Générer `token` -> Email -> Accepter + Payer -> Finaliser (`swap()` Cashier).

8.  **Paiements Mobiles (Capacitor - V1) :** (Standard)
    -   **PAS** d'IAP. On utilise le **Capacitor Browser plugin** pour ouvrir la page de paiement Stripe.

### 6. DevOps (CI/CD - V1)

-   **CI (GitHub Actions) :** Lancer `php artisan test` à chaque `push`.
-   **CD (Ploi.io / Render) :** Déploiement zéro-downtime après CI au vert ✅ sur `main`.

Ceci est notre plan directeur. Je vais maintenant commencer à créer la structure des fichiers et te demander de m'aider à les remplir. Es-tu prêt ?
