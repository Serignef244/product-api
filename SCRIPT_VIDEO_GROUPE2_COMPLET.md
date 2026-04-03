# Script Vidéo de Présentation
## Groupe 2 — Product API | Projet Final DevOps

■ **Durée cible** : 10 à 13 minutes — dans la fourchette imposée (7-15 min)
■ **Date limite** : 08 Avril à 23h59

### Préparation avant de filmer
Avant d'appuyer sur « Enregistrer », vérifiez que tout est prêt :

| Élément | Ce qu'il faut vérifier |
|---|---|
| **VM1 (Ubuntu)** | Terminal ouvert avec `kubectl get pods -n devops-l3gl` — pods en `Running` |
| **VM2 (Kali/Ubuntu)** | Terminal ouvert avec `docker ps` — les 3 conteneurs allumés |
| **Navigateur** | Onglet 1 : Grafana (`http://192.168.239.20:3000`) <br> Onglet 2 : GitLab Pipelines <br> Onglet 3 : Docker Hub |
| **Postman / curl** | Requêtes GET et POST sur `http://192.168.239.10/l3gl/api/products` prêtes à envoyer |
| **Éditeur / GitHub** | Fichiers `k8s/l3gl-devops.yaml`, `prometheus.yml` et `.gitlab-ci.yml` accessibles |

> ■■ **Attention** : Vos IPs sont fixes pour cette présentation : **VM1 = 192.168.239.10** et **VM2 = 192.168.239.20**.

---

### Chronologie globale
| Partie | Contenu | Durée |
|---|---|---|
| **0 — Intro** | Présentation du groupe et du projet | ~1 min |
| **1 — Kubernetes** | Pods, services, ingress, tests API | ~4 min |
| **2 — Monitoring** | Grafana, Prometheus, Loki | ~4 min |
| **3 — CI/CD** | Pipeline GitLab, Docker Hub | ~3 min |
| **4 — Conclusion** | Bilan et difficultés | ~1 min |
| **TOTAL** | | **~13 min** |

---

### Partie 0 — Introduction (~1 minute)
**Ce que vous dites :**
> "Bonjour, nous sommes le Groupe 2. Voici la présentation de notre Projet Final DevOps, qui consiste à déployer notre Product API Spring Boot avec MySQL sur un cluster Kubernetes, le tout automatisé avec GitLab CI/CD et monitoré avec Prometheus, Grafana et Loki."

**Ce que vous montrez :**
- Le schéma d'architecture du projet (depuis Notion ou une slide)
- Les deux VMs dans VirtualBox brièvement

---

### Partie 1 — Kubernetes (~4 minutes)
**Étape 1 : Montrer les ressources Kubernetes**
Tapez ces commandes dans le terminal de la VM1 :
```bash
kubectl get namespaces
kubectl get pods -n devops-l3gl
kubectl get deployment -n devops-l3gl
kubectl get service -n devops-l3gl
kubectl get ingress -n devops-l3gl
```
**Ce que vous dites :**
> "Nous avons déployé notre application dans le namespace `devops-l3gl`. Comme vous le voyez, MySQL et notre Product API sont en statut `Running` et `1/1 Ready`. L'application est exposée à l'intérieur du cluster via un Service `ClusterIP` sur le port `8086`, et accessible depuis l'extérieur grâce à un Ingress contrôlé par un Reverse Proxy Nginx."

**Étape 2 : Tester l'API en direct (obligatoire !)**
Ouvrez Postman ou un terminal (directement sur votre PC ou la VM1) et effectuez ces deux appels :
```bash
# Terminal
curl -X POST http://192.168.239.10/l3gl/api/products -H "Content-Type: application/json" -d '{"name": "Bureau", "price": 150}'
curl -X GET http://192.168.239.10/l3gl/api/products
```
**Ce que vous dites :**
> "Nous allons tester l'API via l'adresse publique de la VM1 (`192.168.239.10`) à travers le Reverse Proxy Nginx. Le POST crée un produit en base MySQL. Le GET nous confirme que la donnée est bien persistée."
■ **Montrez** le code `201 Created` (ou `200`) pour le POST et la liste JSON pour le GET.

**Étape 3 : Montrer les fichiers de configuration**
Affichez le fichier `k8s/l3gl-devops.yaml` dans votre éditeur ou GitHub.
**Ce que vous dites :**
> "Voici notre fichier de déploiement Kubernetes. On y trouve : les limites de ressources CPU (200m/400m) et mémoire (250Mi/500Mi), les sondes Liveness et Readiness sur `/actuator/health/liveness` et `/actuator/health/readiness`, et les variables de configuration montées depuis une ConfigMap."

Montrez aussi :
- `kubectl logs <nom-du-pod> -n devops-l3gl` (logs de l'application)
- Le fichier `k8s/mysql.yaml` (Secret MySQL + Deployment + Service)
- La configuration Nginx sur la VM1 (`/etc/nginx/sites-available/...`)

---

### Partie 2 — Monitoring (~4 minutes)
**Étape 1 : Montrer les conteneurs Docker (VM2)**
Dans le terminal de la VM2, tapez :
```bash
docker ps
```
**Ce que vous dites :**
> "Sur notre VM2, nous avons déployé via Docker Compose trois outils d'observabilité : Prometheus pour la collecte de métriques, Grafana pour la visualisation, et Loki pour la centralisation des logs."
■ On doit voir 3 conteneurs : `prometheus`, `grafana`, `loki` — tous en statut `Up`.

**Étape 2 : Montrer Prometheus**
Ouvrez Prometheus dans le navigateur : `http://192.168.239.20:9090`
**Ce que vous dites :**
> "Prometheus scrape notre application toutes les 10 secondes sur l'endpoint `/actuator/prometheus`. On peut voir ici dans *Status > Targets* que notre Product API est bien en statut UP."

**Étape 3 : Montrer les dashboards Grafana**
Ouvrez Grafana : `http://192.168.239.20:3000`
**Ce que vous dites :**
> "Nous avons créé deux dashboards dans Grafana : un pour compter les appels `GET /api/products`, et un autre pour les appels `POST /api/products`."
■ **Astuce :** faites 3-4 appels GET et POST sur Postman, revenez sur Grafana, actualisez — montrez les compteurs qui augmentent en direct !

**Étape 4 : Montrer les logs Loki**
Dans Grafana, allez dans `Explore` → source `Loki` → tapez la requête : `{app="product-api"}` → `Run Query`.
**Ce que vous dites :**
> "Les logs de notre application Spring Boot remontent automatiquement vers la VM2 (`192.168.239.20`) grâce au LokiAppender configuré dans `logback-spring.xml`. On peut voir ici les requêtes traitées en temps réel."

**Étape 5 : Montrer les fichiers de configuration**
- Fichier `observability/prometheus.yml` → montrez `scrape_interval: 10s` et la target VM1 (`192.168.239.10:8086`)
- Fichier `observability/docker-compose.yml` → montrez les 3 services
- Fichier `src/main/resources/logback-spring.xml` → montrez l'URL Loki `http://192.168.239.20:3100`

---

### Partie 3 — GitLab CI/CD (~3 minutes)
**Étape 1 : Montrer le pipeline GitLab**
Ouvrez GitLab → votre projet → CI/CD > Pipelines. Cliquez sur le dernier pipeline vert.
**Ce que vous dites :**
> "Notre pipeline CI/CD se déclenche automatiquement à chaque push sur la branche `develop`. Il est composé de 3 jobs : test, build et docker."

Montrez chaque job en cliquant dessus :
- **test** (Logs Maven OK) : "Le job test lance les tests unitaires avec Maven."
- **build** (Logs avec BUILD SUCCESS) : "Le job build compile et génère le fichier JAR de l'application."
- **docker** (Logs avec docker push réussi) : "Le job docker construit l'image et la pousse sur Docker Hub automatiquement."

**Étape 2 : Montrer Docker Hub**
Ouvrez votre profil Docker Hub (`serignef244`) dans le navigateur.
**Ce que vous dites :**
> "Voici notre image Docker publiée sur Docker Hub. Le tag `latest` a été mis à jour automatiquement après le passage au vert du pipeline."

**Étape 3 : Montrer le fichier .gitlab-ci.yml**
Affichez le fichier `.gitlab-ci.yml` depuis GitLab ou votre éditeur.
**Ce que vous dites :**
> "Le pipeline utilise Docker-in-Docker pour construire l'image. Les identifiants Docker Hub sont stockés dans les variables secrètes de GitLab (DOCKER_USERNAME, DOCKER_PASSWORD, DOCKER_IMAGE) pour ne jamais exposer de mot de passe dans le code."
■■ *Ne dites pas que vous utilisez H2 pour les tests. Dites simplement : "les tests unitaires passent avec succès".*

---

### Partie 4 — Conclusion (~1 minute)
**Ce que vous dites :**
> "Pour conclure, ce projet nous a permis de déployer une application Spring Boot de bout-en-bout : du code Java jusqu'à l'infrastructure Kubernetes, en passant par le monitoring avec Prometheus et Grafana, et l'automatisation CI/CD avec GitLab. Nous avons rencontré des difficultés — notamment avec le démarrage de l'image Alpine Java, les Liveness Probes Kubernetes qui expiraient trop vite (que nous avons augmenté à `600s`), et la communication réseau entre les deux VMs — que nous avons su surmonter.
> Merci pour votre attention."

---
### Checklist finale avant de filmer
- [ ] Les pods sont en `Running` sur VM1 (`kubectl get pods -n devops-l3gl`)
- [ ] `docker ps` montre 3 conteneurs actifs sur VM2
- [ ] Grafana accessible et dashboards GET/POST visibles (`http://192.168.239.20:3000`)
- [ ] Prometheus → Status > Targets : Product API en `UP`
- [ ] Logs Loki visibles dans Grafana Explore
- [ ] Au moins 1 appel POST et 1 appel GET testés en direct sur `192.168.239.10`
- [ ] Pipeline GitLab vert avec les 3 jobs réussis
- [ ] Image visible sur Docker Hub
- [ ] Fichiers YAML/config affichables rapidement
