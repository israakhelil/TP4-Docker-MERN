# TP4 : Docker et Docker Compose (application MERN)

Cours : Introduction au DevOps, ISG Sousse, 3ème LIG (enseignante : Zaineb Sakhrawi)

## 1. Objectif

Conteneuriser une application (client React/Vite + API Express) avec Docker, puis l'orchestrer avec Docker Compose avec un conteneur MongoDB.

## 2. Structure du projet

```
TP4-Docker-MERN/
├── Client/                (React + Vite)
│   ├── Dockerfile
│   └── .dockerignore
├── Server/                (API Express)
│   ├── Dockerfile
│   └── .dockerignore
├── captures/
├── docker-compose.yml
└── README.md
```

Projet de départ cloné depuis : https://github.com/wiem-kb/MyAppReact.git

## 3. Dockerfiles

### Server/Dockerfile

```dockerfile
FROM node:lts-alpine
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
EXPOSE 9000
ENV MONGO_URI=mongodb://mongodb:27017/mern-db
CMD ["npm", "start"]
```

- `FROM node:lts-alpine` : image de base Node légère.
- `WORKDIR /app` : répertoire de travail dans le conteneur.
- `COPY package*.json` puis `RUN npm install` : installation des dépendances (couche mise en cache).
- `COPY . .` : copie du reste du code.
- `EXPOSE 9000` : port de l'API.
- `ENV MONGO_URI` : URL MongoDB par défaut (redéfinie dans Docker Compose).
- `CMD ["npm", "start"]` : démarrage de l'application.

### Client/Dockerfile

```dockerfile
FROM node:lts-alpine
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
RUN npm run build
RUN npm install -g serve
EXPOSE 3000
CMD ["serve", "-s", "dist", "-l", "3000"]
```

- `RUN npm run build` : build de production avec Vite (dossier `dist`).
- `RUN npm install -g serve` : serveur HTTP léger.
- `CMD ["serve", "-s", "dist", "-l", "3000"]` : sert les fichiers du build sur le port 3000.

### Fichiers .dockerignore

`node_modules` et `npm-debug.log` sont exclus du contexte de build.

## 4. Construction des images

```bash
docker build -t mern-server ./Server
docker build -t mern-client ./Client
docker images
```

![images](captures/01-docker-images.png)

## 5. Exécution manuelle

```bash
docker network create mern-network
docker run -d --name mongodb --network mern-network mongo
docker run -d --name server --network mern-network -p 9000:9000 -e MONGO_URI=mongodb://mongodb:27017/mern-db mern-server
docker run -d --name client --network mern-network -p 3000:3000 mern-client
docker ps
```

![docker ps](captures/02-docker-ps-manuel.png)

Test de l'API :

```bash
curl http://localhost:9000/api/health
```

Réponse : `{"message":"API is running"}`

![curl](captures/03-curl-api.png)

Le client est accessible sur http://localhost:3000.

![client](captures/04-page-client.png)

Nettoyage avant Docker Compose :

```bash
docker rm -f client server mongodb
docker network rm mern-network
```

## 6. Docker Compose

Le fichier `docker-compose.yml` définit trois services :

- `mongodb` : image `mongo`, conteneur nommé `mongodb`, réseau `mern-network`.
- `server` : build depuis `./Server`, port 9000, variable `MONGO_URI`, dépend de `mongodb`.
- `client` : build depuis `./Client`, port 3000, dépend de `server`.

Le réseau `mern-network` utilise le driver `bridge`. Les services communiquent par leur nom (le serveur accède à MongoDB via le hostname `mongodb`).

Lancement :

```bash
docker compose up --build
```

Vérification :

```bash
docker compose ps
curl http://localhost:9000/api/health
```

![compose ps](captures/05-compose-ps.png)

Arrêt :

```bash
docker compose down
```

## 7. Remarques

- L'énoncé donne `docker build -t mern-server ./client` pour le client : c'est une faute de frappe, l'image du client est nommée `mern-client`.
- Les dossiers du dépôt s'appellent `Client` et `Server` (avec majuscule).
- Le serveur Express fourni n'utilise pas encore MongoDB dans son code (pas de `mongoose`). La variable `MONGO_URI` est définie et transmise comme demandé, et le conteneur MongoDB tourne sur le même réseau, prêt à être utilisé.
- `depends_on` garantit l'ordre de démarrage des conteneurs, pas que MongoDB soit prêt à accepter des connexions.

## 8. Conclusion

Ce TP m'a permis de conteneuriser un client React et une API Express, de les relier à MongoDB via un réseau Docker, puis de tout orchestrer avec une seule commande grâce à Docker Compose.
