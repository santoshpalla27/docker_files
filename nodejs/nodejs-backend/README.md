# Node.js Backend Docker Examples

This directory contains a collection of Dockerfiles demonstrating different strategies for containerizing a Node.js backend application. They range from a simple development setup to highly optimized, secure production builds.

## 📂 Dockerfiles Overview

| File | Type | Description | Best For |
|------|------|-------------|----------|
| `Dockerfile-base` | **Basic** | A simple, single-stage Dockerfile. Runs as `root`. | **Development / Learning** only. Not for production. |
| `Dockerfile-multistage` | **Intermediate** | Uses multi-stage builds to separate build dependencies from the runtime. | **General Purpose** production apps. |
| `Dockerfile-prod` | **Advanced** | Production-ready with `dumb-init`, non-root user, and aggressive caching/pruning. | **High-Scale Production** deployments. |
| `Dockerfile-distroless` | **Security** | Uses Google's Distroless base image for a minimal attack surface (no shell, no package manager). | **High Security** environments. |

---

## 🚀 Usage

### Prerequisites
- Docker installed.
- A Node.js application structure (with `package.json`, `index.js` or `dist/main.js`).

### 1. Basic Setup (`Dockerfile-base`)
Simple setup, but results in a larger image and runs as root.
```bash
docker build -f Dockerfile-base -t my-app:base .
docker run -p 3000:3000 my-app:base
```

### 2. Multi-Stage Build (`Dockerfile-multistage`)
Separates the build environment from the runtime environment to save space.
```bash
docker build -f Dockerfile-multistage -t my-app:multistage .
docker run -p 3000:3000 my-app:multistage
```

### 3. Production Ready (`Dockerfile-prod`)
**Recommended for most use cases.** Includes:
- **`dumb-init`**: Handles PID 1 signals correctly.
- **Non-root user**: Runs as `node` for security.
- **Optimization**: Uses `npm ci` and `npm prune --production`.

```bash
docker build -f Dockerfile-prod -t my-app:prod .
docker run -p 3000:3000 my-app:prod
```

### 4. Distroless (`Dockerfile-distroless`)
Extremely small and secure. Contains only the application and its runtime dependencies.
```bash
docker build -f Dockerfile-distroless -t my-app:distroless .
docker run -p 3000:3000 my-app:distroless
```

---

## 🛡️ Best Practices Implemented

1.  **`.dockerignore`**: A `.dockerignore` file is included to prevent `node_modules`, `.git`, and logs from being copied into the image. This speeds up builds and ensures a clean context.
2.  **Version Pinning**: All Dockerfiles use specific Node.js versions (e.g., `node:22-alpine`) to ensure reproducibility.
3.  **Non-Root User**: Production images (`prod`, `multistage`, `distroless`) run as a non-privileged user to mitigate security risks.
4.  **Immutable Installs**: Uses `npm ci` instead of `npm install` for reliable builds.

## ⚠️ Important Notes
- **Port**: These examples expose port `3000`. Adjust the `EXPOSE` instruction if your app runs on a different port.
- **Entrypoint**: `Dockerfile-prod` assumes your built file is at `dist/main.js`. Update the `CMD` if your entry point differs (e.g., `index.js`).

## Critical Trade-offs with Distroless

| Feature | Alpine Image | Distroless Image |
| :--- | :--- | :--- |
| **Shell Access** | Yes (`/bin/sh`) | **No** (Cannot `docker exec -it ... bash`) |
| **Package Manager** | Yes (`apk`) | **No** (Cannot install `vim`, `curl`, etc.) |
| **Attack Surface** | Small | **Tiny** (Zero CVEs usually) |
| **Debugging** | Easy | **Very Hard** (Must use logs or ephemeral containers) |
| **Startup Command** | `npm start` works | Must use `node index.js` |

