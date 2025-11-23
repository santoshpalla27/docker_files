# Node.js Frontend Docker Examples (React + Vite)

This directory contains Dockerfiles for containerizing a React application built with Vite.

## 📂 Dockerfiles Overview

| File | Type | Description | Best For |
|------|------|-------------|----------|
| `Dockerfile-base` | **Development** | Runs the Vite dev server (`npm run dev`). | **Local Development** with hot reload. |
| `Dockerfile-multistage` | **Standard** | Builds the app and serves static files with Nginx. | **General Production**. |
| `Dockerfile-prod` | **Secure** | Runs Nginx as a non-root user. | **Production** with security requirements. |
| `Dockerfile-distroless` | **Unprivileged** | Uses `nginxinc/nginx-unprivileged` for a secure default. | **High Security** environments. |

---

## 🚀 Usage

### Prerequisites
- Docker installed.

### 1. Development (`Dockerfile-base`)
Runs the dev server. Mounts your code for live updates.
```bash
docker build -f Dockerfile-base -t my-app:dev .
docker run -p 5173:5173 -v $(pwd):/app -v /app/node_modules my-app:dev
```

### 2. Standard Production (`Dockerfile-multistage`)
Builds the React app and serves it with Nginx.
```bash
docker build -f Dockerfile-multistage -t my-app:multistage .
docker run -p 80:80 my-app:multistage
```

### 3. Secure Production (`Dockerfile-prod`)
Runs Nginx as a non-root user.
```bash
docker build -f Dockerfile-prod -t my-app:prod .
docker run -p 80:80 my-app:prod
```

### 4. Unprivileged (`Dockerfile-distroless`)
Uses the official unprivileged Nginx image. Note the port is **8080**.
```bash
docker build -f Dockerfile-distroless -t my-app:distroless .
docker run -p 8080:8080 my-app:distroless
```

---

## ⚙️ Configuration

*   **`nginx.conf`**: A custom Nginx configuration is included to handle Single Page Application (SPA) routing (redirecting 404s to `index.html`).
*   **`.dockerignore`**: Ensures `node_modules` and other unnecessary files aren't copied into the build context.

## Critical Trade-offs with Distroless

| Feature | Alpine Image | Distroless Image |
| :--- | :--- | :--- |
| **Shell Access** | Yes (`/bin/sh`) | **No** (Cannot `docker exec -it ... bash`) |
| **Package Manager** | Yes (`apk`) | **No** (Cannot install `vim`, `curl`, etc.) |
| **Attack Surface** | Small | **Tiny** (Zero CVEs usually) |
| **Debugging** | Easy | **Very Hard** (Must use logs or ephemeral containers) |
| **Startup Command** | `npm start` works | Must use `node index.js` |






The reason is security permissions.

Privileged Ports: In Linux, ports below 1024 (like port 80) are "privileged ports." Only the root user can bind to them.
Non-Root User: The nginxinc/nginx-unprivileged image is designed to run as a non-root user (usually user nginx or 101) for better security.
The Conflict: Since the container is running as a non-root user, it cannot listen on port 80. It would get a "Permission Denied" error.
Solution: The standard practice is to use a high port number like 8080 (or 8000, 3000, etc.), which any user can bind to without special permissions.

When you run this container, you can still map it to port 80 on your host machine if you want:

bash
docker run -p 80:8080 my-app:distroless
This maps port 80 on your computer (host) to port 8080 inside the container.
