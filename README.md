### 1\. Roadmap de Développement (Markdown - V6)

_La V1 reste concentrée sur le MVP. Le nouveau "Branding Dynamique" est ajouté à la V2.0, car il utilise les mêmes données que la "Marque Blanche"._

# Roadmap de Développement - Projet "MonReseau" (V6)

## V1.0 : Le MVP (Objectif : 8-9 Semaines)

_Objectif : Lancer un produit fonctionnel avec notifications temps réel, paiement, création de réseau, et discussions._

* * *

### Phase 1 : Fondation Technique & Authentification (Semaines 1-2)

-   \[ \] **Docker :** Mettre en place `docker-compose.yml` (Nginx, Backend, DB, Redis, MailHog, **Reverb**).
    
-   \[ \] **Backend :** Configurer **Laravel Reverb** (le serveur WebSocket).
    
-   \[ \] **Projets :** Initialiser Laravel (`backend`) et Vue.js (`frontend`).
    
-   \[ \] **Base de Données :** Créer les modèles `User`, `Plan`, `Network`.
    
-   \[ \] **Base de Données :** Créer la table pivot `network_user`.
    
-   \[ \] **Base de Données (Préparation V2) :** Ajouter la colonne `branding` (JSON, nullable) à la table `networks`.
    
-   \[ \] **API :** Implémenter **Laravel Sanctum**. Créer les routes `/api/register` et `/api/login`.
    
-   \[ \] **Frontend :** Créer les pages `Login.vue`, `Register.vue` et le store **Pinia** pour l'authentification.
    

* * *

### Phase 2 : Logique SaaS & Paiement (Semaines 3-4)

-   \[ \] **Backend :** Installer et configurer **Laravel Cashier** (Stripe).
    
-   \[ \] **Backend :** Implémenter l'**Architecture Modulaire** (champ `features` JSON sur le modèle `Plan`).
    
-   \[ \] **Frontend :** Créer la page `CreateNetwork.vue` avec le sélecteur de plan et le formulaire **Stripe.js**.
    
-   \[ \] **API :** Créer la route `POST /api/subscribe` (création du `Network` + Abonnement Cashier).
    
-   \[ \] **Frontend :** Portail de facturation (via redirection vers le "Customer Portal" de Stripe/Cashier).
    

* * *

### Phase 3 : Logique Multi-Réseau (Semaine 5)

-   \[ \] **API :** Mettre à jour l'API `/login` pour qu'elle renvoie la liste des `networks` de l'utilisateur.
    
-   \[ \] **Frontend :** Créer la page `SelectNetwork.vue`.
    
-   \[ \] **Frontend :** Implémenter le **Navigation Guard** (Router Vue) pour rediriger si `networks.length > 1`.
    
-   \[ \] **Backend :** Créer le **Middleware** `EnsureUserBelongsToNetwork` pour sécuriser les routes de l'API.
    
-   \[ \] **Frontend :** Implémenter l'intercepteur Axios pour envoyer le header `X-Network-ID` à chaque requête.
    

* * *

### Phase 4 : Fonctionnalités de Base (Semaine 6)

-   \[ \] **Backend :** Créer le modèle `Post` (lié à `user_id` et `network_id`).
    
-   \[ \] **API :** Créer les routes CRUD pour les `Post`.
    
-   \[ \] **Frontend :** Créer le composant `Feed.vue` (fil d'actualité) et le formulaire de création de post.
    
-   \[ \] **Backend :** Implémenter **Laravel Notifications** (canaux `database`, `mail`, et **`broadcast`** via Reverb).
    
-   \[ \] **Backend :** Définir les `routes/channels.php` pour sécuriser les WebSockets.
    
-   \[ \] **Frontend :** Installer et configurer **Laravel Echo** (client WebSocket).
    
-   \[ \] **Frontend :** Écouter les notifications en temps réel et les ajouter au store Pinia (mise à jour de la cloche 🔔 sans rafraîchissement).
    
-   \[ \] **Backend :** Mettre en place les **Queues** (Redis) pour l'envoi d'emails.
    

* * *

### Phase 5 : Solidification (Semaine 7)

-   \[ \] **Backend :** Implémenter le flux sécurisé de **Transfert de Propriété/Facturation** (le flux en 6 étapes).
    
-   \[ \] **Backend :** Ajouter le support pour les paiements **SEPA** (domiciliation) via Cashier.
    
-   \[ \] **Backend/Frontend :** Affiner la gestion des rôles (inviter un 'admin', 'membre').
    
-   \[ \] **Backend :** Mettre en place le **Logging** (configurer Sentry pour la production).
    

* * *

### Phase 6 : Packaging Mobile & Déploiement CI/CD (Semaines 8-9)

-   \[ \] **DevOps :** Configurer l'infrastructure de production (Serveur chez Scaleway/OVH, BDD managée, Bucket S3).
    
-   \[ \] **DevOps :** Configurer **Ploi.io** ou **Render.com** pour le déploiement.
    
-   \[ \] **CI/CD :** Mettre en place **GitHub Actions** pour la **CI** (Intégration Continue).
    
-   \[ \] **CI/CD :** Configurer la **CD** (Déploiement Continu).
    
-   \[ \] **Mobile :** Ajouter **Capacitor** au projet Vue.js.
    
-   \[ \] **Mobile :** Générer les builds iOS et Android et tester sur simulateurs.
    
-   \[ \] **Mobile :** Implémenter la logique de _masquage_ des paiements dans l'app mobile (redirection vers le web).
    
-   \[ \] **LANCEMENT V1.0 !** 🚀
    

* * *

## V2.0 : Évolution "Premium" (Après le lancement)

_Objectif : Augmenter la valeur perçue, justifier les plans supérieurs et améliorer la rétention._

-   \[ \] **Module "Branding Stratifié" (Pro & Enterprise) :**
    
    -   \[ \] **Backend :** Mettre à jour le champ `plans.features` pour inclure `"dynamic_branding": true` ou `"white_label_app": true`.
        
    -   \[ \] **Backend :** Créer une UI "Admin Réseau" pour que les owners puissent remplir leurs données de `branding` (couleurs, logo) dans la table `networks`.
        
    -   \[ \] **Frontend (Plan Pro) :** Implémenter un **"Theme Manager"** dans Vue.
        
        -   \[ \] Quand un utilisateur sélectionne un réseau, vérifier si `dynamic_branding` est actif.
            
        -   \[ \] Si oui, charger les couleurs/logo depuis l'API et les injecter dynamiquement via des **CSS Custom Properties** (`:root { --primary-color: ... }`).
            
    -   \[ \] **DevOps (Plan Enterprise) :**
        
        -   \[ \] Créer le "Script d'Usine" (Build Factory) qui lit ces mêmes données de `branding` pour configurer `capacitor.config.json` et générer les icônes/splash screens.
            
        -   \[ \] Mettre à jour la CI/CD pour permettre la compilation de ces applications en marque blanche.
            
-   \[ \] **Module "Gamification" :**
    
    -   \[ \] Backend : Système de points et de `Badges` (via Events/Listeners).
        
    -   \[ \] Frontend : Interface pour voir ses points et badges (activée par `feature_flag`).
        
-   \[ \] **Module "Notifications Push Mobiles" :**
    
    -   \[ \] Backend : Ajouter le canal `push` (via Firebase) à Laravel Notifications.
        
    -   \[ \] Mobile : Implémenter le plugin **Capacitor Push Notifications**.
        
-   \[ \] **Module "Analytics" :**
    
    -   \[ \] Backend : Créer les **Tâches Planifiées (Cron)** pour générer des rapports.
        
    -   \[ \] Backend : Envoyer des rapports par email via **Notifications Serveur**.
        
    -   \[ \] Frontend : Créer un tableau de bord simple pour les "owners" (activé par `feature_flag`).
