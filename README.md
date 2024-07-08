# Full Stack Web Application Deployment
Table of Contents
Introduction
Prerequisites
Forking the Repository
Dockerization
Frontend Dockerfile
Backend Dockerfile
Proxy Configuration
Traefik Configuration
Nginx Proxy Manager Configuration
Database Configuration
Adminer Setup
Proxy Manager Setup
Cloud Deployment
Additional Instructions

# Introduction
This document provides detailed instructions for deploying a full stack web application consisting of a React frontend and a FastAPI + PostgreSQL backend using Docker containers. The guide includes steps for configuring a proxy to serve both services on the same port using Traefik or Nginx Proxy Manager and instructions for deploying the application to an AWS EC2 instance.

# Prerequisites
Docker and Docker Compose installed on your local machine
AWS account with permissions to create and manage EC2 instances
Domain name or subdomain from Afraid DNS for your application
Basic knowledge of Docker, Docker Compose, and web proxies
# Forking the Repository
Fork the provided repository to your GitHub account.
Clone the forked repository to your local machine:
bash
Copy code
git clone https://github.com/<your-username>/repository-name.git
cd repository-name
# Dockerization
Frontend Dockerfile
Create a Dockerfile in the frontend directory with the following content:

dockerfile
Copy code
# Stage 1: Build
FROM node:14 AS build

WORKDIR /app
COPY package.json yarn.lock ./
RUN yarn install
COPY . .
RUN yarn build

# Stage 2: Serve
FROM nginx:alpine
COPY --from=build /app/build /usr/share/nginx/html
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
Backend Dockerfile
Create a Dockerfile in the backend directory with the following content:

dockerfile
Copy code
FROM python:3.9-slim

WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY . .
EXPOSE 8000
CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000"]
# Proxy Configuration
# Traefik Configuration
Create a docker-compose.yml file in the root directory with the following content:
yaml
Copy code
version: '3.8'

services:
  frontend:
    build:
      context: ./frontend
    container_name: frontend
    labels:
      - "traefik.http.routers.frontend.rule=PathPrefix(`/`)"
      - "traefik.port=80"

  backend:
    build:
      context: ./backend
    container_name: backend
    labels:
      - "traefik.http.routers.backend.rule=PathPrefix(`/api`, `/docs`, `/redoc`)"
      - "traefik.port=8000"

  traefik:
    image: traefik:v2.5
    container_name: traefik
    ports:
      - "80:80"
      - "8080:8080"
    command:
      - "--api.insecure=true"
      - "--providers.docker=true"
      - "--entrypoints.web.address=:80"
    volumes:
      - "/var/run/docker.sock:/var/run/docker.sock"
# Nginx Proxy Manager Configuration
Install Nginx Proxy Manager by following this guide.
Configure the proxy settings to serve the frontend and backend on the same port:
Frontend: /
Backend: /api, /docs, /redoc
Database Configuration
Create a docker-compose.yml file in the root directory with the following content:

yaml
Copy code
version: '3.8'

services:
  db:
    image: postgres:13
    container_name: postgres
    environment:
      POSTGRES_USER: user
      POSTGRES_PASSWORD: password
      POSTGRES_DB: database
    volumes:
      - postgres_data:/var/lib/postgresql/data

volumes:
  postgres_data:
Ensure your FastAPI application is configured to connect to the PostgreSQL database using the following environment variables:

env
Copy code
DATABASE_URL=postgresql://user:password@db:5432/database
# Adminer Setup
Add Adminer service to your docker-compose.yml file:

yaml
Copy code
services:
  adminer:
    image: adminer
    container_name: adminer
    ports:
      - "8080:8080"
Configure Adminer to be accessible via the subdomain db.domain.

# Proxy Manager Setup
Add Nginx Proxy Manager service to your docker-compose.yml file:

yaml
Copy code
services:
  nginx-proxy-manager:
    image: jc21/nginx-proxy-manager:latest
    container_name: nginx-proxy-manager
    ports:
      - "80:80"
      - "81:81"
      - "443:443"
    environment:
      DB_SQLITE_FILE: "/data/database.sqlite"
    volumes:
      - ./data:/data
      - ./letsencrypt:/etc/letsencrypt
Configure Nginx Proxy Manager to be accessible via the subdomain proxy.domain.

# Cloud Deployment
Create an AWS EC2 instance.
Install Docker and Docker Compose on the EC2 instance.
Clone your forked repository to the EC2 instance.
Deploy the Dockerized application:
bash
Copy code
cd repository-name
docker-compose up -d
# Additional Instructions
Configure HTTP to redirect to HTTPS.
Configure www to redirect to non-www.
Ensure all services are properly configured and running.
Test the application to verify that the frontend, backend, and database are functioning correctly.
By following this guide, you will successfully deploy a full stack web application using Docker containers and proxies. This comprehensive approach ensures a robust and scalable deployment, demonstrating your expertise in DevOps and containerization.

# Good Luck








