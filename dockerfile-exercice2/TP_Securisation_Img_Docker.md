# TP : Sécurisation d'une image Docker

## Mission 1 : Audit de l'image existante (`quiz-docker:initial`)

### 1. État des lieux
* **Image de base :** `node:22.23.5-alpine3.24@sha256:1314fb30ec37592d5845e5a160d522c0e680972aa534549940b8d73bde88481a`
* **Utilisateur :** `root`
```bash
docker exec -it quiz-docker sh
/ $ whoami
root
```
* **Taille totale :** `320MB`
```bash
docker images quiz-docker:initial --format "{{.Size}}"
320MB
```

### 2. Analyse des couches (`docker history`)
```bash
docker history quiz-docker:initial
IMAGE          CREATED       CREATED BY                                      SIZE      COMMENT
e787944646e4   2 days ago    CMD ["serve" "-s" "dist"]                       0B        buildkit.dockerfile.v0
<missing>      2 days ago    EXPOSE [3000/tcp]                               0B        buildkit.dockerfile.v0
<missing>      2 days ago    RUN /bin/sh -c npm run build && rm -rf node_…   81.9kB    buildkit.dockerfile.v0
<missing>      2 days ago    COPY . . # buildkit                             16.9MB    buildkit.dockerfile.v0
<missing>      3 days ago    RUN /bin/sh -c npm ci # buildkit                28.8MB    buildkit.dockerfile.v0
<missing>      3 days ago    COPY package*.json ./ # buildkit                28.7kB    buildkit.dockerfile.v0
<missing>      3 days ago    RUN /bin/sh -c npm install -g serve # buildk…   18.9MB    buildkit.dockerfile.v0
<missing>      3 days ago    WORKDIR /app                                    8.19kB    buildkit.dockerfile.v0
<missing>      4 days ago    CMD ["node"]                                    0B        buildkit.dockerfile.v0
<missing>      4 days ago    ENTRYPOINT ["docker-entrypoint.sh"]             0B        buildkit.dockerfile.v0
<missing>      4 days ago    COPY docker-entrypoint.sh /usr/local/bin/ # …   20.5kB    buildkit.dockerfile.v0
<missing>      4 days ago    RUN /bin/sh -c apk add --no-cache --virtual …   5.49MB    buildkit.dockerfile.v0
<missing>      4 days ago    ENV YARN_VERSION=1.22.22                        0B        buildkit.dockerfile.v0
<missing>      4 days ago    RUN /bin/sh -c addgroup -g 1000 node     && …   158MB     buildkit.dockerfile.v0
<missing>      4 days ago    ENV NODE_VERSION=22.23.5                        0B        buildkit.dockerfile.v0
<missing>      10 days ago   CMD ["/bin/sh"]                                 0B        buildkit.dockerfile.v0
<missing>      10 days ago   ADD alpine-minirootfs-3.24.4-aarch64.tar.gz …   9.31MB    buildkit.dockerfile.v0
```

### 3. Analyses de sécurité

* **Hadolint :**
```bash
hadolint Dockerfile
Dockerfile:7 DL3016 warning: Pin versions in npm. Instead of `npm install <package>` use `npm install <package>@<version>`
```

* **Trivy (vulnérabilités) :**
```bash
trivy image -q --severity HIGH,CRITICAL quiz-docker:initial

Report Summary
┌───────────────────────────────────────┬──────────┬─────────────────┬─────────┐
│                Target                 │   Type   │ Vulnerabilities │ Secrets │
├───────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ quiz-docker:initial (alpine 3.24.4)   │  alpine  │        0        │    -    │
│ app/package.json                      │ node-pkg │        0        │    -    │
│ opt/yarn-v1.22.22/package.json        │ node-pkg │        0        │    -    │
│ usr/local/lib/node_modules/...        │ node-pkg │       22        │    -    │
└───────────────────────────────────────┴──────────┴─────────────────┴─────────┘
```

* **Dockle :**
```bash
dockle quiz-docker:initial
WARN    - CIS-DI-0001: Create a user for the container (* Last user should not be root)
INFO    - CIS-DI-0005: Enable Content trust for Docker
INFO    - CIS-DI-0006: Add HEALTHCHECK instruction to the container image
INFO    - DKL-LI-0003: Only put necessary files (* Suspicious directory : root/.npm)
```

### 4. Analyse et risques identifiés
* **Adaptée à la production :** Non, exécution en root, taille excessive de 320 MB et présence d'outils de compilation inutiles au runtime.
* **Composants nécessaires :** Les fichiers statiques du dossier `dist` et un serveur web léger. L'environnement Node.js et npm est superflu.
* **Risques identifiés :** Élévation de privilèges via root, surface d'attaque étendue avec shell et gestionnaire de paquets, risque de fuite de secrets sans `.dockerignore`.
* **Améliorations prioritaires :** Multi-stage build, utilisateur non-root, serveur web dédié et fichier `.dockerignore`.

---

## Mission 2 : Sécurisation et Remédiation

### 1. Fichier `.dockerignore`
```text
Dockerfile*
.gitignore
*.md
.env
*.log
.git
node_modules
.vscode
dist
```

### 2. Dockerfile multi-stage (`quiz-docker:secure`)
```dockerfile
ARG NODE_VERSION=22.23.5-alpine3.24@sha256:1314fb30ec37592d5845e5a160d522c0e680972aa534549940b8d73bde88481a
ARG NGINX_VERSION=1.31.8-alpine3.24-slim@sha256:b19233c62309c8e620732244f9d0d0008678854cb9c6c47898817e8dec93a9e3

# Étape 1 : Build
FROM node:${NODE_VERSION} AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

# Étape 2 : Production
FROM nginx/nginx-unprivileged:${NGINX_VERSION} AS serve
COPY --from=builder /app/dist /usr/share/nginx/html
EXPOSE 8080
CMD ["nginx", "-g", "daemon off;"]
```

### 3. Exécution sécurisée
```bash
docker run -d \
  --name quiz-docker-secure \
  -p 8082:8080 \
  --memory="256m" \
  --cpus="0.5" \
  --security-opt no-new-privileges:true \
  --cap-drop ALL \
  quiz-docker:secure
```

---

## Mission 3 : Résultats et Comparatif

### 1. Validation de la remédiation

* **Utilisateur non-root (`nginx`) :**
```bash
docker exec -it quiz-docker-secure whoami
nginx
```

* **Scan Trivy (0 vulnérabilité) :**
```bash
trivy image -q --severity HIGH,CRITICAL quiz-docker:secure

Report Summary
┌───────────────────────────────────────┬────────┬─────────────────┬─────────┐
│                Target                 │  Type  │ Vulnerabilities │ Secrets │
├───────────────────────────────────────┼────────┼─────────────────┼─────────┤
│ quiz-docker:secure (alpine 3.24.4)    │ alpine │        0        │    -    │
└───────────────────────────────────────┴────────┴─────────────────┴─────────┘
```

* **Réduction de la taille :**
```bash
docker images quiz-docker:initial --format "{{.Size}}"
320MB

docker images quiz-docker:secure --format "{{.Size}}"
21.7MB
```

### 2. Tableau comparatif

| Critère | Initial (`quiz-docker:initial`) | Sécurisé (`quiz-docker:secure`) |
| :--- | :--- | :--- |
| **Utilisateur** | `root` (UID 0) | `nginx` |
| **Taille disque** | **320 MB** | **21.7 MB** (-93%) |
| **Taille DockerHub** | ~140 MB | **~6.1 MB** |
| **Architecture** | Mono-stage (Node + npm + dev) | Multi-stage (Nginx slim non root) |
| **Vulnérabilités High/Crit** | 22 failles npm | **0** |
| **`.dockerignore`** | Absent | Présent |
| **Limitation ressources** | Aucune | 256MB RAM / 0.5 CPU / Cap-Drop |

### 3. Publication sur Docker Hub

```bash
docker tag quiz-docker:secure remyduboisit/quiz-docker:secure
docker push remyduboisit/quiz-docker:secure
```

* **Image publique :** `docker.io/remyduboisit/quiz-docker:secure`
* **URL DockerHub :** `https://hub.docker.com/repository/docker/remyduboisit/quiz-docker/tags/secure`

### 4. Analyse des résultats
* **Améliorations les plus significatives :** Le multi-stage build avec Nginx slim non-root supprime 100% des vulnérabilités npm et réduit la taille de 93%.
* **Diminution de la taille :** Le SDK Node, le cache npm et les dépendances de build sont exclus de l'image finale au profit du dossier `dist`.
* **Dockerfile vs exécution :** Le Dockerfile gère le multi-stage, l'utilisateur non-root et les images minimales. L'exécution gère les quotas de ressources et les restrictions de privilèges kernel.
* **Sécurité totale :** Non, de nouvelles vulnérabilités peuvent être découvertes.
* **Mesures complémentaires en production :** Montage du système de fichiers en lecture seule et intégration des scans dans la CI/CD.



