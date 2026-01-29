# explanation.md

## 1. Choice of Base Images

- **Backend:** We use `node:18-alpine` for a lightweight, fast Node.js environment. It reduces image size and speeds up builds.
- **Frontend:** We use a two-stage build:
  - Stage 1: `node:18` to build the React app.
  - Stage 2: `nginx:alpine` to serve the static build files via Nginx (small and production-ready).

## 2. Dockerfile Directives

### Backend Dockerfile
- `WORKDIR /app` sets the working directory.
- `COPY . .` copies all files.
- `RUN npm install` installs dependencies.
- `CMD ["npm", "start"]` starts the server.

### Frontend Dockerfile
- Build stage:
  - `RUN npm install && npm run build`
- Serve stage:
  - `COPY build/ /usr/share/nginx/html` moves build to Nginx default directory.

## 3. Docker-Compose Networking

- Compose creates a **bridge network**  (`yolo_net`) that connects frontend and backend services.
- Port mappings:
  - `3000:80` maps Nginx (port 80) to host port 3000.
  - `5000:5000` maps backend API to host port 5000.
- Frontend can access backend via service name `yolo-backend` internally.

## 4. Volumes

- MongoDB uses a persistent volume `mongo_data` mounted at `/data/db` inside the container.
- This ensures data persists across container restarts and cleanups.
- Volume driver is set to `local` for local storage on the host machine.
- Environment variable `MONGO_URI=mongodb://mongo:27017/yolodb` connects the backend to MongoDB via the internal Docker network.

## 5. Git Workflow

- The project follows a simple Git workflow:
  - `master` is the stable branch.
  - Changes and Dockerfile updates are committed progressively.
  - Image tags are based on Git updates and commits (e.g., `v1.0.0`).

## 6. Application Running & Debugging

- The app runs successfully using:
  ```bash
  docker compose up -d
  ```
