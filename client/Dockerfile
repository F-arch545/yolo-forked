# client/Dockerfile

# Stage 1: Build stage using lightweight Node.js 20

FROM node:18-alpine AS build

WORKDIR /usr/src/app

# Copy package files and install dependencies

COPY package*.json ./
RUN npm install

# Copy the rest of the application

COPY . .

# Stage 2: Runtime stage

FROM node:18-alpine

WORKDIR /app

# Copy build files from the previous stage

COPY --from=build /usr/src/app /app

ENV NODE_OPTIONS=--openssl-legacy-provider
# Expose the frontend port

EXPOSE 3000

# Start the React development server

CMD ["npm", "start"]

# Build the application
