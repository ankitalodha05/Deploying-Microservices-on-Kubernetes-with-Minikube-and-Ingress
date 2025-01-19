# Dockerfile
FROM node:16-alpine

# Set working directory
WORKDIR /app

# Copy package.json and package-lock.json
COPY package.json ./

# Install dependencies
RUN npm install

# Copy the rest of the application code
COPY . .

# Expose the service port
EXPOSE 3001

# Start the service
CMD ["node", "app.js"]
