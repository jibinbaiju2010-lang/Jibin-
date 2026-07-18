# Jibin-
Leads management platform for multinational companies and local business 
version: '3.8'

services:
  postgres:
    image: pgvector/pgvector:pg16
    container_name: jerome_db
    environment:
      POSTGRES_USER: jerome
      POSTGRES_PASSWORD: "secure_password_change_me"
      POSTGRES_DB: jerome_leads
    volumes:
      - jerome_pg_data:/var/lib/postgresql/data
    ports:
      - "5432:5432"
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U jerome"]
      interval: 10s
      timeout: 5s
      retries: 5

  backend:
    build: ./backend
    container_name: jerome_api
    environment:
      - DB_HOST=postgres
      - DB_PORT=5432
      - DB_USER=jerome
      - DB_PASSWORD=secure_password_change_me
      - DB_NAME=jerome_leads
      - JWT_SECRET=super_secret_jwt_key_min_32_chars_long
      - APP_ENV=development
      - PORT=8080
    ports:
      - "8080:8080"
    depends_on:
      postgres:
        condition: service_healthy
    volumes:
      - ./backend:/app

  frontend:
    build: ./frontend
    container_name: jerome_web
    ports:
      - "4200:4200"
    volumes:
      - ./frontend:/app
      - /app/node_modules
    command: npm run start -- --host 0.0.0.0 --poll 2000
    depends_on:
      - backend

  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf:ro
    depends_on:
      - backend
      - frontend
    profiles:
      - prod

volumes:
  jerome_pg_data:
  services:
  - type: web
    name: jerome-api
    runtime: docker
    repo: https://github.com/YOUR_USERNAME/jerome-platform
    dockerfilePath: ./backend/Dockerfile
    envVars:
      - key: DB_HOST
        value: YOUR_SUPABASE_HOST
      - key: DB_PORT
        value: 5432
      - key: DB_USER
        value: postgres
      - key: DB_PASSWORD
        value: YOUR_SUPABASE_PASSWORD
      - key: DB_NAME
        value: postgres
      - key: JWT_SECRET
        generateValue: true
      - key: APP_ENV
        value: production
      - key: PORT
        value: 8080
    healthCheckPath: /health

  - type: static
    name: jerome-web
    runtime: static
    buildCommand: cd frontend && npm install && npm run build -- --configuration=production
    publishDir: frontend/dist/jerome/browser
