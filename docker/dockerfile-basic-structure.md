# Use the official Node.js image as the base
FROM node:22-alpine

# Set working directory inside the container
WORKDIR /app

# Copy package.json into the container
COPY package*.json ./

# Install dependencies (Use npm ci for consistent installs)
RUN npm ci

# Copy the application code into the container
COPY . .

# Expose port 5000 (tells Docker this container listens on 5000)
EXPOSE 5000

# Run the application
CMD [ "npm", "run", "dev" ]
