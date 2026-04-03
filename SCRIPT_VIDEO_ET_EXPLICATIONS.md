# Guide Complet : Présentation Vidéo (Projet DevOps)

Ce document est votre script pas-à-pas pour réussir la vidéo de présentation exigée par votre professeur (qui doit durer entre 7 et 15 minutes). L'astuce est de montrer d'abord le résultat visuel, puis les fichiers de code qui ont permis ce résultat.

## Préparation avant de filmer 🎬
1. Ouvrez toutes vos fenêtres : 
   - Votre navigateur web avec **Grafana** (`http://192.168.239.20:3000`), **GitLab** (page Pipelines), et **Docker Hub** (votre profil serignef244) ouverts sur des onglets différents.
   - Votre terminal de la **VM1 (Ubuntu)** avec `kubectl get pods -n devops-l3gl` déjà prêt et tapé (vérifiez bien qu'ils sont `1/1 Running`).
   - Votre terminal de la **VM2** avec `docker ps` déjà tapé (les 3 conteneurs allumés).
   - Un terminal ou Postman pour tester l'API (`curl`).
2. Ouvrez votre éditeur de code ou l'interface web GitLab/GitHub pour montrer les fichiers YAML très facilement.

---

## 🎬 SCRIPT DE LA VIDÉO (Chronologie exacte)

### 1. Introduction (Environ 1 minute)
> *"Bonjour, nous sommes le Groupe 2. Voici la présentation de notre Projet Final DevOps, qui consiste à automatiser et déployer notre Product API avec une base de données MySQL sur un cluster Kubernetes, le tout automatisé avec CI/CD et monitoré de bout-en-bout."*
- Affichez brièvement l'architecture globale du projet (le schéma conceptuel si vous l'avez sur Notion).

### 2. Partie 1 : Kubernetes & Architecture (Environ 4 minutes)
**Ce qu'il faut dire et montrer :**
1. **Montrez le Terminal de la VM1 :**
   - *"Nous avons déployé notre application dans l'environnement Minikube dans le namespace `devops-l3gl`."*
   - Tapez (ou montrez) `kubectl get pods -n devops-l3gl`.
   - *"Comme vous le voyez, MySQL et notre API sont en statut 'Running' et '1/1 Ready'."*
   - Tapez `kubectl get ingress -n devops-l3gl`.
   - *"Notre application est contenue au sein du cluster (via ClusterIP) et exposée vers l'extérieur uniquement via notre Ingress, lui-même contrôlé par un Reverse Proxy Nginx sur le port 80."*
2. **Test de l'application (Démonstration en direct) :**
   - Ouvrez Postman (ou votre terminal de test).
   - *"Nous allons maintenant tester l'API Product en passant par l'adresse de notre Reverse Proxy."*
   - Envoyez la requête `POST http://localhost/l3gl/api/products` (ou l'IP de la VM si testé depuis Windows) avec un JSON (ex: `{"name": "Bureau", "price": 150}`).
   - Montrez que le code "201 Created" (ou 200) s'affiche avec les détails du produit.
   - Envoyez la requête `GET http://localhost/l3gl/api/products`.
   - Montrez que la liste des produits est bien renvoyée en entier (ce qui prouve que la base MySQL stocke bien les données !).
3. **Explication du K8s (Le Code) :**
   - Affichez le fichier `k8s/l3gl-devops.yaml` à l'écran. 
   - *"Voici le cœur de notre configuration. Vous remarquerez que nous avons :*
     - *Défini des limites de ressources (250Mi à 500Mi en RAM, et 200m à 400m en CPU) pour empêcher l'API d'asphyxier la machine virtuelle.*
     - *Configuré une sonde Liveness et Readiness sur `/actuator/health` sur le port `8086`. Le délai a été adapté (240s) pour contourner la saturation.*
     - *Lié la variable LOKI_URL avec l'IP exacte de notre VM2 (`192.168.239.20`)."*

### 3. Partie 2 : Monitoring / Observabilité sur VM2 (Environ 4 minutes)
**Ce qu'il faut dire et montrer :**
1. **Montrez le Terminal de la VM2 :**
   - *"Pour l'observabilité, nous avons alloué une seconde VM (VM2). Nous y avons lancé Prometheus, Grafana et Loki via Docker Compose."*
   - Tapez `docker ps` pour montrer que les 3 conteneurs tournent.
2. **Montrez Grafana (Navigateur Web) :**
   - Allez sur la page internet de Grafana (`http://192.168.239.20:3000`).
   - *"Nous avons connecté Grafana à Prometheus et Loki. Voici nos deux tableaux de bords qui surveillent nos endpoints : un pour compter les appels GET, un autre pour les appels POST."*
   - **Astuce :** Refaites 3 ou 4 appels `GET` et `POST` sur Postman. Retournez sur Grafana, cliquez sur le bouton "Actualiser", et **montrez au professeur que les compteurs ont augmenté en direct !**
3. **Montrez Loki (Les Logs de Java) :**
   - Dans Grafana, allez dans l'onglet **Explore** (à gauche), choisissez la source `Loki`, vérifiez la ligne `{app="product-api"}` et choisissez "Run Query".
   - *"Comme vous le constatez, les logs de notre application Java (Spring Boot) remontent automatiquement par le réseau jusqu'à la VM2 grâce à notre configuration LokiAppender."*
4. **Explication du Monitoring (Le Code) :**
   - Montrez le fichier `prometheus.yml`.
   - *"Nous avons configuré un paramètre 'scrape_interval' de 10 secondes et nous scrutons l'adresse IP de notre VM1 (`192.168.239.10:8086`) pour aspirer les métriques Actuator."*

### 4. Partie 3 : CI/CD avec GitLab (Environ 3 minutes)
**Ce qu'il faut dire et montrer :**
1. **Montrez GitLab (Navigateur Web) :**
   - Allez sur la page GitLab (Build > Pipelines). Cliquez sur le dernier pipeline vert.
   - *"Nous avons mis en place une intégration et un déploiement continus total. À chaque 'push' sur la branche `develop`, GitLab lance un système d'usine complet décomposé en 3 Jobs."*
   - Cliquez brièvement sur chaque bulle verte :
     - *"L'étape `test` vérifie que le code Java compile bien avec Maven. Nous avons utilisé H2 en base in-memory pour que le test passe sans MySQL."*
     - *"L'étape `build` génère l'artefact .jar de l'application."*
     - *"L'étape `docker` fabrique l'image JRE et l'envoie automatiquement sur notre registre Docker Hub."*
2. **Montrez le Docker Hub (Navigateur Web) :**
   - Allez sur votre profil public Docker Hub.
   - *"Vous pouvez voir ici que la dernière image 'latest' a été poussée juste après le passage au vert du pipeline."*
3. **Explication de la CI/CD (Le Code) :**
   - Affichez le fichier `.gitlab-ci.yml`.
   - *"Le pipeline utilise le Docker in Docker (dind) et se sert des variables secrètes configurées dans notre administration GitLab pour les identifiants Docker Hub, afin de garantir une sécurité totale (aucun mot de passe dans le code)."*

### 5. Conclusion (Moins d'une minute)
> *"Pour conclure, ce projet nous a permis d'interconnecter de bout-en-bout le code Java, l'infrastructure locale, le cloud Docker Hub et l'usine GitLab. Nous avons rencontré et surmonté d'importants défis, comme le gel silencieux du démarrage Java sur Alpine que nous avons résolu en utilisant le moteur `jammy`, ou encore la saturation RAM et les Timeouts Kubernetes traités avec une politique imagePullPolicy et des Delays adaptés.*
> *Merci pour votre attention !"*

---

🎉 **Si vous suivez précisément cet ordre en montrant ces écrans, vous gagnerez l'intégralité des points validés par le professeur (1 point vidéo vérifiable) !**
