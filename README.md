# MERN Docker Compose Application

This repository contains a MERN stack application containerized with Docker. It includes manual setup for the frontend, backend, and database, and uses Docker Compose for management.

## Manual Containerization

### Step 1: Create a Common Network

Create a common network named `mern` that will be used by all services:

```bash
docker network create mern
```

### Frontend

1. **Dockerfile in `frontend`**:
    ```dockerfile
    FROM node:18.18.0
    WORKDIR /app
    COPY package.json ./
    RUN npm install
    COPY . .
    EXPOSE 5173
    CMD ["npm", "run", "dev"]
    ```

2. **Build and run**:
    ```bash
    cd frontend
    docker build -t mern-frontend .
    docker run --name frontend --network=mern -d -p 5173:5173 mern-frontend
    ```

### Database

Run MongoDB:
```bash
docker run --name=mongodb --network=mern -d -p 27017:27017 -v ~/opt/data:/data/db mongo:latest
```
    

### Backend

1. **Dockerfile in `backend`**:
    ```dockerfile
    FROM node:18.18.0
    WORKDIR /app
    COPY package.json ./
    RUN npm install
    COPY . .
    EXPOSE 5000
    CMD ["npm", "start"]
    ```

2. **Build and run**:
    ```bash
    cd backend
    docker build -t mern-backend .
    docker run --name backend --network=mern -d -p 5000:5000 mern-backend
    ```

## Using Docker Compose

1. **Create `docker-compose.yml`**:
    ```yaml
    version: '3.8'
    services:
      mongodb:
        image: mongo:latest  
        ports:
          - "27017:27017"  
        networks:
          - mern_network
        volumes:
          - mongo-data:/data/db  

      backend:
        build: ./mern/backend
        ports:
          - "5050:5050" 
        networks:
          - mern_network
        environment:
          MONGO_URI: mongodb://mongo:27017/mydatabase  
        depends_on:
          - mongodb

      frontend:
        build: ./mern/frontend
        ports:
          - "5173:5173"  
        networks:
          - mern_network
        environment:
          REACT_APP_API_URL: http://backend:5050 

    networks:
      mern_network:
        driver: bridge 

    volumes:
      mongo-data:
        driver: local  # Persist MongoDB data locally
    ```

2. **Run with Docker Compose**:
    ```bash
    docker-compose up
    ```

This configuration will set up your MongoDB, backend, and frontend services with networking and data persistence.

Hence Dockerized !
