# Stage 1: Build dependencies
# Use lightweight Node.js base image  (smaller file size)

FROM node:18-alpine AS build

# Set working directory

WORKDIR /usr/src/app

# Copy only the package files 

COPY package*.json ./

# Install only production dependencies

RUN npm install --only=production


# Stage 2: Final slim image for running app

FROM node:18-alpine

# Set working directory

WORKDIR /usr/src/app

# Copy only the built node_modules from the build stage

COPY --from=build /usr/src/app/node_modules ./node_modules

# Copy only necessary app files from your local project (NOT everything)

COPY . .

# Expose the backend port

EXPOSE 5000

# Start the app

CMD ["npm", "start"]