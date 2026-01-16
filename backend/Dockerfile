# Dockerfile for Node.js backend application
# Use lightweight Node.js base image  (smaller file size)

FROM node:18-alpine

# Set working directory

WORKDIR /usr/src/app

# Copy package files

COPY package*.json ./

# Install dependencies

RUN npm install

# Copy the entire project

COPY . .

# Expose the backend port

EXPOSE 5000

# Start the app

CMD ["npm", "start"]
