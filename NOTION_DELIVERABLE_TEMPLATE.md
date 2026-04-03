# Projet Final DevOps - Product API
**Groupe 2 : [Vos Noms ICI]**
**Date : Avril 2026**

---

## 1. Architecture Kubernetes (VM1)

### Description
Notre application Spring Boot et sa base de données MySQL ont été déployées sur un cluster **Minikube** (VM1). L'exposition vers l'extérieur est assurée par un **Ingress Nginx** et un **Reverse Proxy Nginx** configuré sur l'hôte Ubuntu.

### Captures d'écran (À insérer)
> [!TIP]
> *Insérez ici :*
> 1. Capture de `kubectl get pods -n devops-l3gl` (Statut Running)
> 2. Capture de `kubectl get ingress -n devops-l3gl` (Adresse IP visible)
> 3. Capture du test API réussi : `curl -X GET http://localhost/l3gl/api/products`

### Fichiers de Configuration

#### `mysql.yaml`
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: mysql-secret
  namespace: devops-l3gl
type: Opaque
data:
  MYSQL_ROOT_PASSWORD: cm9vdHBhc3N3b3Jk # rootpassword
  MYSQL_PASSWORD: cHJvZHVjdHBhc3N3b3Jk # productpassword
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mysql-pvc
  namespace: devops-l3gl
spec:
  accessModes: ["ReadWriteOnce"]
  resources:
    requests:
      storage: 1Gi
---
# [Le reste du fichier mysql.yaml...]
```

#### `l3gl-devops.yaml`
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: product-api
  namespace: devops-l3gl
spec:
  # ... (Configs de ressources et sondes)
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: product-api-ingress
  namespace: devops-l3gl
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /$2
    nginx.ingress.kubernetes.io/use-regex: "true"
spec:
  rules:
    - http:
        paths:
          - path: /l3gl(/|$)(.*)
            pathType: ImplementationSpecific
            backend:
              service:
                name: product-api
                port:
                  number: 8086
```

---

## 2. Monitoring & Observabilité (VM2)

### Description
Le monitoring est centralisé sur la **VM2** via une stack **Docker Compose**. Nous collectons les métriques avec **Prometheus**, visualisons les données avec **Grafana** et centralisons les logs avec **Loki**.

### Captures d'écran (À insérer)
> [!TIP]
> *Insérez ici :*
> 1. Capture de `docker ps` sur VM2 (Les 3 conteneurs Up)
> 2. Capture de la page Prometheus `Targets` (Statut UP en vert)
> 3. Capture du Dashboard Grafana avec les courbes GET/POST
> 4. Capture des logs Loki dans Grafana Explore (`{app="product-api"}`)

### Fichiers de Configuration

#### `docker-compose.yml`
```yaml
version: '3.8'
services:
  prometheus:
    image: prom/prometheus:latest
    ports: ["9090:9090"]
  loki:
    image: grafana/loki:latest
    ports: ["3100:3100"]
  grafana:
    image: grafana/grafana:latest
    ports: ["3000:3000"]
```

#### `prometheus.yml`
```yaml
global:
  scrape_interval: 10s
scrape_configs:
  - job_name: 'product-api'
    metrics_path: '/l3gl/actuator/prometheus'
    static_configs:
      - targets: ['192.168.239.10:80']
```

---

## 3. Pipeline CI/CD (GitLab)

### Description
L'automatisation est gérée par **GitLab CI**. Chaque modification sur la branche `develop` déclenche un pipeline complet :
1. **Test** : Validation des tests unitaires (Maven).
2. **Build** : Compilation et génération de l'artefact JAR.
3. **Docker** : Création de l'image Docker et push sur Docker Hub.

### Variables CI/CD utilisées
- `DOCKER_USERNAME` : Identifiant Docker Hub.
- `DOCKER_PASSWORD` : Jeton d'accès Docker Hub.
- `DOCKER_IMAGE` : Nom de l'image (`serignef244/product-api`).

### Captures d'écran (À insérer)
> [!TIP]
> *Insérez ici :*
> 1. Capture de la vue Pipelines GitLab (Tout en vert)
> 2. Capture des logs du job `docker` (Push réussi)
> 3. Capture de l'image publiée sur Docker Hub

#### `.gitlab-ci.yml`
```yaml
stages:
  - test
  - build
  - docker

test:
  image: maven:3.8-openjdk-17
  script: mvn clean test

build:
  image: maven:3.8-openjdk-17
  script: mvn clean package -DskipTests
  artifacts:
    paths: [target/*.jar]

docker:
  image: docker:latest
  services: [docker:dind]
  script:
    - docker login -u $DOCKER_USERNAME -p $DOCKER_PASSWORD
    - docker build -t $DOCKER_IMAGE:latest .
    - docker push $DOCKER_IMAGE:latest
```

---

## 4. Conclusion & Difficultés
Nous avons réussi à mettre en œuvre une chaîne DevOps complète. Les principales difficultés ont été :
- La gestion des ressources RAM/CPU limitées sur les VMs (correction via l'augmentation des délais des Sondes Kubernetes à 600s).
- La configuration complexe de l'Ingress Nginx pour les réécritures de chemins (correction via l'annotation `use-regex`).
- La communication réseau entre VM1 et VM2 pour le monitoring.
