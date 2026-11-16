ARG NODE_VERSION=22.23.5-alpine3.24@sha256:1314fb30ec37592d5845e5a160d522c0e680972aa534549940b8d73bde88481a

# Utiliser une image de base Node.js légère
FROM node:${NODE_VERSION}

# Définir le répertoire de travail de l'application
WORKDIR /app

# Copier les fichiers package.json et package-lock.json pour installer les dépendances
COPY package*.json ./

# Copier les fichiers sources de l'application dans le conteneur
COPY src/ ./src/
COPY public/ ./public/

# Installer les dépendances, construire l'application et nettoyer les modules Node.js
RUN npm ci && npm run build && npm install -g serve && rm -rf node_modules

# Exposer le port
EXPOSE 3000

# Démarrer l'application 
CMD ["serve", "-s", "build"]