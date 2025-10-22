# Roadmap de Développement - Projet "MonReseau" (V7)

## V1.0 : Le MVP (Objectif : 8-9 Semaines)

### 

_Objectif : Lancer une plateforme **communautaire** fonctionnelle. Créer la fondation (le "contenant") avant de prouver le ROI (le "contenu")._

_(Pas de changement sur le MVP, on reste concentré.)_

* * *

### Phase 1 : Fondation Technique & Authentification (Semaines 1-2)

### 

-   \[ \] **Docker :** `docker-compose.yml` (Nginx, Backend, DB, Redis, MailHog, **Reverb**).
    
-   \[ \] **Backend :** Configurer **Laravel Reverb**.
    
-   \[ \] **Projets :** Initialiser Laravel (`backend`) et Vue.js (`frontend`).
    
-   \[ \] **Base de Données :** Créer les modèles `User`, `Plan`, `Network`.
    
-   \[ \] **Base de Données :** Créer la table pivot `network_user`.
    
-   \[ \] **Base de Données (Préparation V2) :** Ajouter la colonne `branding` (JSON, nullable) à la table `networks`.
    

* * *

### Phase 2 : Logique SaaS & Paiement (Semaines 3-4)

### 

-   \[ \] **Backend :** Installer **Laravel Cashier** (Stripe).
    
-   \[ \] **Backend :** Implémenter le champ `features` (JSON) sur le modèle `Plan`.
    
-   \[ \] **Frontend :** Créer la page `CreateNetwork.vue` (Sélecteur de plan + Formulaire Stripe.js).
    
-   \[ \] **API :** Créer la route `POST /api/subscribe`.
    
-   \[ \] **Frontend :** Portail de facturation (Redirection "Customer Portal" de Stripe).
    

* * *

### Phase 3 : Logique Multi-Réseau (Semaine 5)

### 

-   \[ \] **API :** Mettre à jour l'API `/login` (renvoyer `user.networks[]`).
    
-   \[ \] **Frontend :** Créer la page `SelectNetwork.vue`.
    
-   \[ \] **Frontend :** Implémenter le **Navigation Guard** (Router Vue) pour la sélection de réseau.
    
-   \[ \] **Backend :** Créer le **Middleware** `EnsureUserBelongsToNetwork`.
    
-   \[ \] **Frontend :** Implémenter l'intercepteur Axios (header `X-Network-ID`).
    

* * *

### Phase 4 : Fonctionnalités Communautaires (Semaine 6)

### 

-   \[ \] **Backend :** Créer le modèle `Post`.
    
-   \[ \] **API :** Créer les routes CRUD pour les `Post`.
    
-   \[ \] **Frontend :** Créer le composant `Feed.vue`.
    
-   \[ \] **Backend :** Implémenter **Laravel Notifications** (canaux `database`, `mail`, **`broadcast`**).
    
-   \[ \] **Backend :** Définir les `routes/channels.php` (sécurisation WebSocket).
    
-   \[ \] **Frontend :** Installer et configurer **Laravel Echo** (client WebSocket).
    
-   \[ \] **Frontend :** Implémenter la cloche 🔔 de notification en temps réel.
    
-   \[ \] **Backend :** Mettre en place les **Queues** (Redis) pour l'envoi d'emails.
    

* * *

### Phase 5 : Solidification (Semaine 7)

### 

-   \[ \] **Backend :** Implémenter le flux sécurisé de **Transfert de Propriété/Facturation**.
    
-   \[ \] **Backend :** Ajouter le support **SEPA** (domiciliation) via Cashier.
    
-   \[ \] **Backend/Frontend :** Gestion des rôles (inviter 'admin', 'membre').
    
-   \[ \] **Backend :** Mettre en place le **Logging** (Sentry).
    

* * *

### Phase 6 : Packaging Mobile & Déploiement CI/CD (Semaines 8-9)

### 

-   \[ \] **DevOps :** Configurer l'infrastructure de production (Serveur, BDD Managée, Bucket S3).
    
-   \[ \] **DevOps :** Configurer **Ploi.io** ou **Render.com**.
    
-   \[ \] **CI/CD :** Mettre en place **GitHub Actions** pour la **CI** (tests auto).
    
-   \[ \] **CI/CD :** Configurer la **CD** (déploiement auto).
    
-   \[ \] **Mobile :** Ajouter **Capacitor** au projet Vue.js.
    
-   \[ \] **Mobile :** Générer les builds iOS et Android (tests simulateurs).
    
-   \[ \] **Mobile :** Implémenter la logique de _masquage_ des paiements.
    
-   \[ \] **LANCEMENT V1.0 !** 🚀
    

* * *

## V2.0 : Évolution "Business & Premium" (Après le lancement)

### 

_Objectif : Prouver le ROI du réseau et augmenter la valeur des plans supérieurs._

-   \[ \] **Module "Opportunity Tracker" (La Killer Feature) :**
    
    -   \[ \] **Backend :** Créer la table `opportunities` (`network_id`, `from_user_id`, `to_user_id`, `description`, `contact_name`, `status`, `value`).
        
    -   \[ \] **Backend :** Créer les routes API CRUD pour les opportunités (sécurisées).
        
    -   \[ \] **Backend :** Activer la fonctionnalité via le `feature_flag` "opportunity\_tracker".
        
    -   \[ \] **Backend :** Envoyer une notification (via Reverb) lors de la création.
        
    -   \[ \] **Frontend :** Créer la page "Opportunités" (vues "Reçues" / "Envoyées").
        
    -   \[ \] **Frontend :** Créer le formulaire de création d'opportunité.
        
    -   \[ \] **Frontend :** Permettre au destinataire de changer le statut (`Gagné`/`Perdu`) et d'ajouter la `value`.
        
    -   \[ \] **Backend/Frontend :** Créer le **Dashboard Admin** qui agrège le `SUM(value)` pour le chef de réseau.
        
-   \[ \] **Module "Branding Stratifié" (Pro & Enterprise) :**
    
    -   \[ \] **Backend :Un** `feature_flag` "dynamic\_branding" / "white\_label\_app".
        
    -   \[ \] **Frontend (Plan Pro) :** Implémenter un **"Theme Manager"** qui applique le branding (CSS Custom Properties) dynamiquement.
        
    -   \[ \] **DevOps (Plan Enterprise) :** Créer le "Script d'Usine" (Build Factory) pour les applications en marque blanche.
        
-   \[ \] **Module "Gamification" :**
    
    -   \[ \] Backend : Système de points et de `Badges` (ex: "Meilleur apporteur d'affaires").
        
-   \[ \] **Module "Notifications Push Mobiles" :**
    
    -   \[ \] Backend : Ajouter le canal `push` (Firebase) à Laravel Notifications.
        
    -   \[ \] Mobile : Implémenter le plugin **Capacitor Push Notifications**.
        
-   \[ \] **Module "Analytics" :**
    
    -   \[ \] Backend : Tâches planifiées (Cron) pour rapports d'engagement.
