# 🛍️ Yolo E-Commerce Web App

Yolo is a full-stack e-commerce platform built with the **MERN stack** and fully containerized using **Docker** and **Docker Compose** for easy deployment and development.

---

## 🧰 Stack Overview

- **Frontend**: React.js (served via Nginx)
- **Backend**: Node.js + Express
- **Database**: MongoDB (Local containerized)
- **Containerization**: Docker & Docker Compose

---

## 📁 Project Layout

```
yolo-forked/
├── backend/              # Express API + MongoDB Models
│   ├── Dockerfile
│   ├── package.json
│   ├── server.js         # Main server file
│   ├── models/
│   │   └── Products.js
│   └── routes/
│       └── api/
│           └── productRoute.js
├── client/               # React app
│   ├── Dockerfile
│   ├── nginx.conf        # Nginx configuration for SPA
│   ├── package.json
│   ├── public/
│   └── src/
│       ├── components/
│       └── images/
├── docker-compose.yaml   # Container orchestration
├── explanation.md        # Docker & deployment details
└── README.md
```

---

## 🚀 Quick Start

1. **Clone the repo:**

```bash
git clone https://github.com/kadimasum/yolo.git
cd yolo-forked
```

2. **Run the app with Docker Compose:**

```bash
docker compose up -d
```

The app will automatically:
- Build the backend and frontend images
- Start MongoDB with persistent volume
- Set up internal networking between containers

---

## 🌐 Access Points

| Service        | URL                                |
|----------------|------------------------------------|
| Frontend       | http://localhost:3000              |
| Backend API    | http://localhost:5000              |
| API Test Route | http://localhost:5000/api          |
| Products API   | http://localhost:5000/api/products |
| MongoDB        | mongodb://localhost:27017/yolodb   |

---

## 🌐 Live Deployment

The application is live at: [http://a2f94f894be434d4c8f57340f59bc977-1852385997.us-east-1.elb.amazonaws.com](http://a2f94f894be434d4c8f57340f59bc977-1852385997.us-east-1.elb.amazonaws.com)

---

## 🛠️ Docker Details

### Containers

- **yolo-client** - React frontend served via Nginx (port 3000 → 80)
- **yolo-backend** - Node.js/Express API (port 5000)
- **yolo-mongo** - MongoDB database (port 27017)

### Docker Images

- `riunguflo/yolo-client:v1.0.0`
- `riunguflo/yolo-backend:v1.0.0`

### Network

- **yolo-net** - Custom bridge network for inter-container communication
  - Subnet: 172.20.0.0/16
  - Allows services to communicate via service names (e.g., `yolo-backend:5000`)

---

## 💾 Volumes

- **mongo_data** - Persistent volume for MongoDB data at `/data/db`
  - Ensures data persists across container restarts
  - Driver: local

---

## 🔧 Environment Setup

The backend connects to MongoDB using the `MONGO_URI` environment variable set in docker-compose.yaml:

```yaml
environment:
  - MONGO_URI=mongodb://mongo:27017/yolodb
```

- Default database: `yolodb`
- MongoDB service name: `mongo` (resolved internally via Docker network)

---

## 📸 Published Docker Images

Docker images are published to Docker Hub under the `riunguflo` account:

```bash
# Pull images directly
docker pull riunguflo/yolo-client:v1.0.0
docker pull riunguflo/yolo-backend:v1.0.0
```

---

## 📤 Build & Deployment Workflow

### Build Images Locally

```bash
docker build -t riunguflo/yolo-backend:v1.0.0 ./backend
docker build -t riunguflo/yolo-client:v1.0.0 ./client
```

### Login to Docker Hub

```bash
docker login
```

### Push to Docker Hub

```bash
docker push riunguflo/yolo-backend:v1.0.0
docker push riunguflo/yolo-client:v1.0.0
```

---

## 📝 Backend Details

### Server (Node.js + Express)

- **Port**: 5000
- **Main File**: `backend/server.js`
- **Database**: MongoDB (Mongoose ODM)
- **Middleware**: CORS, JSON body parser, multer for file uploads

### Available Routes

- `GET /` - Welcome message
- `GET /api` - API status check
- `GET/POST /api/products` - Product CRUD operations

---

## 🎨 Frontend Details

### Client (React + Nginx)

- **Port**: 3000 (maps to 80 internally via Nginx)
- **Build Tool**: React Scripts
- **Proxy**: Configured to forward requests to backend at `http://localhost:5000`
- **Features**:
  - Product listing and detail views
  - Add/Edit product forms
  - Navigation and footer components

### Nginx Configuration

- Serves static React build files
- Handles SPA routing (rewrites to index.html)
- Configured via `client/nginx.conf`

---

## ✅ Git Workflow

```bash
# Stage changes
git add .

# Commit with message
git commit -m "Your message"

# Push to remote
git push origin master
```

---

## 🧪 Troubleshooting

### Container won't start?

```bash
# Check logs
docker compose logs yolo-backend
docker compose logs yolo-client
docker compose logs yolo-mongo

# Rebuild and restart
docker compose down
docker compose up --build
```

### MongoDB connection issues?

Ensure the `MONGO_URI` matches the service name in docker-compose.yaml (`mongodb://mongo:27017/yolodb`)

### Frontend can't reach backend?

Check that the backend proxy in `client/package.json` is set to `"http://localhost:5000"`

---

## 👨‍💻 Author

Built by **Flozy Irungu**

---

## 📄 License

MIT
