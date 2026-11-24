TP4
Structure du projet
TP4-Docker-MERN/
├── Client/
│   ├── Dockerfile
│   └── .dockerignore
├── Server/
│   ├── Dockerfile
│   └── .dockerignore
├── captures/
├── docker-compose.yml
└── README.md

Dockerfile du serveur
dockerfile
FROM node:lts-alpine
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
EXPOSE 9000
ENV MONGO_URI=mongodb://mongodb:27017/mern-db
CMD ["npm", "start"]

Dockerfile du client
dockerfile
FROM node:lts-alpine
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
RUN npm run build
RUN npm install -g serve
EXPOSE 3000
CMD ["serve", "-s", "dist", "-l", "3000"]

Construction des images

docker build -t mern-server ./Server
docker build -t mern-client ./Client
docker images



Exécution manuelle

docker network create mern-network
docker run -d --name mongodb --network mern-network mongo
docker run -d --name server --network mern-network -p 9000:9000 -e MONGO_URI=mongodb://mongodb:27017/mern-db mern-server
docker run -d --name client --network mern-network -p 3000:3000 mern-client
docker ps

docker ps

curl http://localhost:9000/api/health

curl

Client : http://localhost:3000

client

Docker Compose

docker compose up --build
docker compose ps
docker compose down
