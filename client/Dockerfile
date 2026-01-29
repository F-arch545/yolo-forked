# -------------------------------------------------------
# Stage 1: Build the React app using Node.js
# -------------------------------------------------------
# Use lightweight Node.js base image (smaller file size)

FROM node:18-alpine AS build

# Set working directory

WORKDIR /app

# Install dependencies

COPY package*.json ./
RUN npm install

# Copy source files and build

COPY . .
RUN NODE_OPTIONS=--openssl-legacy-provider npm run build

# -------------------------------------------------------
# Stage 2: Serve with NGINX (Production)
# -------------------------------------------------------

FROM nginx:alpine

# Clean default nginx static files

RUN rm -rf /usr/share/nginx/html/*

# Copy built React app from builder stage

COPY --from=build /app/build /usr/share/nginx/html

# Optional: Add custom nginx config for SPA routing

COPY nginx.conf /etc/nginx/conf.d/default.conf

# Expose port 80

EXPOSE 80

# Run nginx in the foreground

CMD ["nginx", "-g", "daemon off;"]