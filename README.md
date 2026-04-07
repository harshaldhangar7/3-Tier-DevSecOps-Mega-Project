# 3-Tier DevSecOps Mega Project

Full-stack user management application with:

- `client/`: React frontend
- `api/`: Node.js + Express REST API
- `MySQL`: persistent data store

The app supports:

- user registration and login
- JWT-based authentication
- role-aware access (`admin`, `viewer`)
- user CRUD from the dashboard
- local development and containerized deployment

## Architecture

```text
React Client (3000 or NGINX:80)
        |
        | HTTP /api/*
        v
Node.js Express API (5000)
        |
        | MySQL connection
        v
MySQL Database
```

## Project Structure

```text
.
|-- api/
|   |-- app.js
|   |-- controllers/
|   |-- middleware/
|   |-- models/
|   `-- routes/
|-- client/
|   |-- src/
|   |-- public/
|   |-- Dockerfile
|   `-- default.conf
`-- README.md
```

## Features

- Register a new user
- Login with email and password
- Seed a default admin user on API startup
- View all users after authentication
- Add users from the dashboard
- Update and delete users as admin
- Frontend routing with login, register, dashboard, and not found pages

## Tech Stack

### Frontend

- React
- React Router
- Axios
- CSS

### Backend

- Node.js
- Express
- MySQL2
- JSON Web Token
- bcryptjs
- dotenv

### Containerization

- Docker
- NGINX for frontend static hosting

## Prerequisites

Install these before running the project:

- Node.js `18+` recommended for local development
- npm
- MySQL `8+` recommended
- Docker optional, for containerized runs

## Environment Variables

This repo uses separate environment files for frontend and backend.

### Backend env: `api/.env`

Required variables:

```env
DB_HOST=localhost
DB_USER=root
DB_PASSWORD=your_mysql_password
DB_NAME=crud_app
JWT_SECRET=replace_with_a_long_random_secret
```

Optional admin seed variables:

```env
ADMIN_NAME=Admin User
ADMIN_EMAIL=admin@example.com
ADMIN_PASSWORD=admin123
ADMIN_ROLE=admin
PORT=5000
```

What they do:

- `DB_HOST`: MySQL server host
- `DB_USER`: MySQL username
- `DB_PASSWORD`: MySQL password
- `DB_NAME`: database name used by the API
- `JWT_SECRET`: secret used to sign JWT tokens
- `ADMIN_*`: seeded admin account created if it does not already exist
- `PORT`: backend listening port, default is `5000`

### Frontend env: `client/.env`

```env
REACT_APP_API=http://localhost:5000
```

Notes:

- The frontend automatically appends `/api` internally.
- If you set `REACT_APP_API=http://localhost:5000`, API requests go to `http://localhost:5000/api/...`.
- If you set `REACT_APP_API=https://your-domain.com/api`, it will still work.

## Recommended Env Setups

### 1. Local development on one machine

Use this when React, API, and MySQL all run locally.

`api/.env`

```env
DB_HOST=localhost
DB_USER=root
DB_PASSWORD=your_password
DB_NAME=crud_app
JWT_SECRET=dev_local_secret_change_me
ADMIN_NAME=Admin User
ADMIN_EMAIL=admin@example.com
ADMIN_PASSWORD=admin123
ADMIN_ROLE=admin
PORT=5000
```

`client/.env`

```env
REACT_APP_API=http://localhost:5000
```

### 2. Frontend and backend on different hosts

Use this when React is deployed separately and calls a remote API.

`client/.env`

```env
REACT_APP_API=https://api.your-domain.com
```

If your backend is already exposed under `/api`, this also works:

```env
REACT_APP_API=https://api.your-domain.com/api
```

### 3. Docker or Kubernetes style ingress

Use this when NGINX, ingress, or a gateway forwards `/api` traffic to the backend.

`client/.env`

```env
REACT_APP_API=https://your-domain.com
```

Or:

```env
REACT_APP_API=https://your-domain.com/api
```

### 4. Staging environment

`api/.env`

```env
DB_HOST=staging-mysql
DB_USER=staging_user
DB_PASSWORD=strong_staging_password
DB_NAME=crud_app
JWT_SECRET=staging_secret_change_me
ADMIN_NAME=Staging Admin
ADMIN_EMAIL=admin-staging@example.com
ADMIN_PASSWORD=change_me_now
ADMIN_ROLE=admin
PORT=5000
```

`client/.env`

```env
REACT_APP_API=https://staging.example.com
```

### 5. Production environment

`api/.env`

```env
DB_HOST=prod-mysql
DB_USER=prod_user
DB_PASSWORD=very_strong_password
DB_NAME=crud_app
JWT_SECRET=very_long_random_production_secret
ADMIN_NAME=Platform Admin
ADMIN_EMAIL=admin@example.com
ADMIN_PASSWORD=change_this_immediately
ADMIN_ROLE=admin
PORT=5000
```

`client/.env`

```env
REACT_APP_API=https://app.example.com
```

Production tips:

- never commit real `.env` files
- use a strong `JWT_SECRET`
- rotate seeded admin credentials after first login
- restrict database access to trusted networks only
- terminate TLS at ingress, reverse proxy, or load balancer

## Database Setup

Create the database first:

```sql
CREATE DATABASE crud_app;
```

Then create the `users` table:

```sql
USE crud_app;

CREATE TABLE users (
  id INT AUTO_INCREMENT PRIMARY KEY,
  name VARCHAR(100) NOT NULL,
  email VARCHAR(150) NOT NULL UNIQUE,
  password VARCHAR(255) NOT NULL,
  role VARCHAR(20) NOT NULL DEFAULT 'viewer',
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

Important:

- the API waits for a `users` table before startup completes
- the API seeds an admin user automatically if `ADMIN_EMAIL` does not already exist

## Local Run Guide

### 1. Install dependencies

Backend:

```bash
cd api
npm install
```

Frontend:

```bash
cd client
npm install
```

### 2. Configure environment files

Create:

- `api/.env`
- `client/.env`

Use the examples from the sections above.

### 3. Start MySQL

Make sure MySQL is running and the database/table already exist.

### 4. Start the backend

```bash
cd api
npm start
```

Expected backend behavior:

- waits until the `users` table is available
- seeds the admin user if missing
- starts on `http://localhost:5000`

### 5. Start the frontend

Open a new terminal:

```bash
cd client
npm start
```

Frontend runs at:

- `http://localhost:3000`

## Default Login

If the admin seed variables are configured, use:

- Email: value of `ADMIN_EMAIL`
- Password: value of `ADMIN_PASSWORD`

Example:

```text
Email: admin@example.com
Password: admin123
```

Change these defaults in non-demo environments.

## API Routes

Base path: `/api`

### Auth

- `POST /api/auth/register`
- `POST /api/auth/login`

### Users

- `GET /api/users`
- `POST /api/users`
- `PUT /api/users/:id`
- `DELETE /api/users/:id`

## Auth Behavior

- JWT token is returned on successful login
- frontend stores token and user info in `localStorage`
- protected requests send `Authorization: Bearer <token>`
- user list requires authentication
- update and delete require admin role

## Docker Support

This repo includes Dockerfiles for both frontend and backend.

### Backend image

Build:

```bash
docker build -t user-manager-api ./api
```

Run:

```bash
docker run --name user-manager-api ^
  -p 5000:5000 ^
  --env-file api/.env ^
  user-manager-api
```

### Frontend image

Build:

```bash
docker build -t user-manager-client --build-arg REACT_APP_API=http://localhost:5000 ./client
```

Run:

```bash
docker run --name user-manager-client -p 3000:80 user-manager-client
```

Notes:

- the frontend Docker image builds static files and serves them with NGINX
- `client/default.conf` proxies `/api/` to `backend-svc:5000`
- if you are not using a service named `backend-svc`, update `client/default.conf` for your environment

## Running In Different Environments

### Windows

- use PowerShell or Command Prompt
- if MySQL runs locally, keep `DB_HOST=localhost`
- use separate terminals for API and client

### Linux

- use shell terminals for API and client
- ensure MySQL user has permission on `crud_app`
- if you use a remote DB, update `DB_HOST`

### macOS

- same flow as Linux
- ensure local MySQL is running before starting the API

### VM, cloud, or server deployment

- configure `api/.env` with the real DB host and credentials
- expose backend port `5000` or reverse-proxy it
- build frontend with the correct `REACT_APP_API`
- use HTTPS in staging and production

## Common Problems

### Registration fails

Check:

- MySQL is running
- `crud_app.users` table exists
- `api/.env` has the right DB credentials
- frontend `REACT_APP_API` points to the backend host

### Login stays on the same page

Check:

- backend is reachable
- `REACT_APP_API` is correct
- browser dev tools show no failed `/api/auth/login` request
- backend returns a valid token and user object

### Dashboard does not load users

Check:

- token exists in localStorage
- `Authorization` header is being sent
- backend JWT secret is the same for login and protected routes

### Docker frontend cannot reach backend

Check:

- `client/default.conf` upstream target
- backend container name or service name
- exposed ports and network configuration

## Useful Commands

Frontend:

```bash
cd client
npm start
npm run build
npm test
```

Backend:

```bash
cd api
npm start
```

## Security Notes

- replace default admin credentials before real use
- use strong secrets in all non-local environments
- do not commit `.env` files with real credentials
- use HTTPS outside local development
- limit database exposure to trusted services only

## Suggested Next Improvements

- add `docker-compose.yml` for one-command startup
- add `.env.example` files in `api/` and `client/`
- add database migration or initialization scripts
- add health check endpoints
- add refresh-token or session expiration handling
- add backend validation and better error responses

## License

See [LICENSE](f:/3-Tier-DevSecOps-Mega-Project/LICENSE).
