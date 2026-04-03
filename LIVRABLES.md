# Livrables : Projet Final DevOps (Product API)

Ce fichier regroupe tous les éléments de configuration externes, les scripts de commandes et les documents requis pour le projet.

## 1. Configuration Nginx (Reverse Proxy) - VM1
Fichier à placer dans `/etc/nginx/sites-available/l3gl-proxy` :
```nginx
server {
    listen 80;
    server_name _; # Ou remplacer par <IP-VM1>

    location /l3gl/ {
        # Proxy vers l'Ingress Kubernetes (Minikube).
        proxy_pass http://<IP-MINIKUBE>:80;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```
Activer la configuration : 
```bash
sudo ln -s /etc/nginx/sites-available/l3gl-proxy /etc/nginx/sites-enabled/
sudo systemctl restart nginx
```

## 2. Déploiement Kubernetes - VM1
Commandes pour appliquer et vérifier les configurations Kubernetes :
```bash
# Vérifier si Minikube est lancé
minikube status

# Créer l'ingress addon si ce n'est pas déjà fait
minikube addons enable ingress

# Appliquer la base de données (Namespace intégré)
kubectl apply -f k8s/mysql.yaml

# Appliquer l'application Product API
kubectl apply -f k8s/l3gl-devops.yaml

# Vérifications
kubectl get namespaces
kubectl get pods -n devops-l3gl
kubectl get deployment -n devops-l3gl
kubectl get service -n devops-l3gl
kubectl get ingress -n devops-l3gl
kubectl logs <nom-du-pod-product-api> -n devops-l3gl
```

## 3. Monitoring et Observabilité - VM2
Commandes de lancement :
```bash
# Aller dans le dossier observability
cd observability

# Lancer la stack avec Docker Compose
docker compose up -d

# Vérifier les conteneurs actifs
docker ps
```
### Accès et Configuration
- **Grafana** : `http://<IP-VM2>:3000` (identifiants standards : admin/admin). Ajouter Prometheus (`http://prometheus:9090`) et Loki (`http://loki:3100`) en Datasources.
- **Requêtes PromQL pour les Dashboards** :
  - **Dashboard 1 (Nombre d'appels GET)** : `http_server_requests_seconds_count{method="GET", uri="/api/products"}`
  - **Dashboard 2 (Nombre d'appels POST)** : `http_server_requests_seconds_count{method="POST", uri="/api/products"}`

## 4. Script de Démonstration Vidéo (7 à 15 minutes)

**Introduction (30s) :**
- Présentation courte du binôme (Groupe 2).
- Rappel du but : Déploiement Product API (Spring Boot + MySQL) sur Kubernetes (Minikube), CI/CD sur GitLab et Monitoring (Prometheus/Grafana/Loki).

**Partie Kubernetes (3 min) :**
- Montrer l'exécution des commandes : `kubectl get pods -n devops-l3gl`, `kubectl get svc`, `kubectl get ingress`.
- Expliquer brièvement les fichiers `mysql.yaml` et `l3gl-devops.yaml` (montrer la LivenessProbe et les Limites CPU/RAM).
- Démontrer l'accès externe via le proxy Nginx (`http://<IP-VM1>/l3gl/api/products`).

**Partie Tests API (2 min) :**
- Ouvrir Postman ou utiliser Curl.
- Effectuer une requête `POST /api/products` (avec un JSON body) -> Montrer le code de retour 200/201.
- Effectuer une requête `GET /api/products` -> Montrer le résultat JSON retourné.

**Partie Monitoring (3 min) :**
- Montrer l'interface Prometheus (Targets -> product-api doit être en "UP").
- Ouvrir Grafana. Montrer les deux Dashboards spécifiques avec le nombre de GET et POST qui augmente lorsque l'API est appelée.
- Ouvrir l'onglet Explore de Grafana et montrer les logs de l'API (label `app=product-api`) qui remontent correctement via Loki.

**Partie GitLab CI/CD (2 min) :**
- Montrer le fichier `.gitlab-ci.yml` et les variables CI configurées dans les paramètres GitLab de votre repo.
- Lancer (ou montrer) un pipeline complet exécuté avec succès avec les 3 stages (test, build, docker).
- Montrer l'image Docker de l'API fraîchement poussée et disponible sur votre profil Docker Hub.

**Conclusion (1 min) :**
- Bilan : validation du fonctionnement global, et indiquer les éventuels défis rencontrés et comment vous les avez surmontés.

## 5. Template Notion Public
Structure à copier-coller pour votre rendu Notion final :

```text
# Projet Final DevOps - Groupe 2 (Product API)
Binôme : [Prénom1 Nom1], [Prénom2 Nom2]
Email : [Vos e-mails]
Lien vidéo de démo : [Lien YouTube/Drive]
Lien GitHub (code source public) : [Lien Repo] 
Lien GitLab (CI/CD) : [Lien Repo] 
Lien Docker Hub (image publiée) : [Lien]

---

## Partie 1 : Architecture & Kubernetes (VM1)
L'API Product (sur le port 8086) est connectée à une BD MySQL et déployée sur un cluster Minikube dans le Namespace `devops-l3gl`. L'accès externe se fait via le proxy Nginx (http://<IP-VM1>/l3gl/api/products).

[Capture d'écran de `kubectl get namespaces` et `kubectl get pods`]
[Capture d'écran de `kubectl get ingress`]
[Capture d'écran du test de l'API (GET et POST) via Postman/Curl]

**Fichiers de configuration K8s :**
[Insérer ici un bloc de code avec le contenu de /k8s/mysql.yaml]
[Insérer ici un bloc de code avec le contenu de /k8s/l3gl-devops.yaml]

---

## Partie 2 : Monitoring (VM2)
L'observabilité de l'API est assurée par Prometheus (qui scrape `/actuator/prometheus` toutes les 10s), Grafana et Loki (via `loki-logback-appender` dans Spring Boot).

[Capture d'écran des Datasources Prometheus et Loki connectées dans Grafana]
[Capture d'écran de votre Dashboard 1 : GET /api/products]
[Capture d'écran de votre Dashboard 2 : POST /api/products]
[Capture d'écran des Logs de Spring Boot visibles dans Grafana / Explore (Loki)]

**Fichiers de configuration Observabilité :**
[Insérer ici un bloc de code avec le contenu de /observability/docker-compose.yml]
[Insérer ici un bloc de code avec le contenu de /observability/prometheus.yml]

---

## Partie 3 : Intégration Continue (GitLab CI/CD)
Le pipeline s'exécute automatiquement après chaque push sur la branche 'develop'. Il intègre trois jobs successifs : tests unitaires, build via Maven et création/push de l'image Docker.

[Capture d'écran du pipeline complet réussi sur la console GitLab]

**Fichier de pipeline :**
[Insérer ici un bloc de code avec le contenu du fichier .gitlab-ci.yml à la racine]
```
