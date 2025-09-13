# Base image
FROM node:18-alpine

# Set working directory
WORKDIR /usr/src/app

# Copy package.json and package-lock.json first
COPY package*.json ./

# Install dependencies
RUN npm install --only=production

# Copy all source code
COPY . .

# Expose app port (change if your app uses different port)
EXPOSE 8080

# Start the app
CMD ["npm", "start"]
