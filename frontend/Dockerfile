# Use official Node.js runtime as base image
FROM node:18-alpine

# Set working directory in container
WORKDIR /app

# Copy the entire frontend application
COPY . .

# Expose port 3000
EXPOSE 3000

# Install a simple HTTP server to serve static files
RUN npm install -g http-server

# Serve the frontend files from templates directory with single-page app fallback
CMD ["http-server", "templates", "-p", "3000", "--cors", "--gzip"]
