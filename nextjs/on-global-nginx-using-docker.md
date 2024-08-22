# How to Deploy a Next.js Project on Nginx Using Docker

Welcome to the deployment guide for deploying your Next.js project using Docker and Nginx. This comprehensive guide is designed to help you efficiently deploy your Next.js application in a production environment. By following these step-by-step instructions, you'll ensure that your web application is robust, scalable, and ready to handle production-level traffic.

## Prerequisites

Before you begin, make sure you have the following:

- A working Next.js project.
- Docker and Docker Compose installed on your server.
- Nginx installed globally on your server.
- Basic knowledge of Linux server administration.
- Access to a domain or subdomain for your Next.js application.

Now, let's dive into the deployment process and get your Next.js project up and running in a production environment!

## Setting Up the Project

Follow these steps to prepare your Next.js project for deployment by containerizing it using Docker and setting up necessary environment variables.

1. **Install Dependencies:**
   Ensure your Next.js project has all the necessary dependencies installed. Run:

   ```sh
   npm install
   ```

2. **Create a `.env` File:**
   Add a `.env` file in the root of your project to store environment variables:

   ```sh
   # .env
   NEXT_PUBLIC_API_URL=https://your.api.url
   ```

   Adjust the variables according to your project's needs.

3. **Update the `package.json` Scripts:**
   Modify the `package.json` scripts to include a build script for production:

   ```json
   {
     "scripts": {
       "dev": "next dev",
       "build": "next build",
       "start": "next start"
     }
   }
   ```

4. **Build Your Project:**
   Run the following command to build your Next.js application for production:

   ```sh
   npm run build
   ```

## Setting Up Docker

5. **Create a `Dockerfile`:**
   In the root of your project, create a `Dockerfile` to define your Docker image:

   ```Dockerfile
   # Dockerfile

   # Use an official Node.js image as the base image
   FROM node:18-alpine

   # Set the working directory in the container
   WORKDIR /app

   # Copy the package.json and package-lock.json files to the container
   COPY package*.json ./

   # Install dependencies
   RUN npm install --production

   # Copy the rest of the application code to the container
   COPY . .

   # Build the Next.js app
   RUN npm run build

   # Expose the port the app runs on
   EXPOSE 3000

   # Start the application
   CMD ["npm", "run", "start"]
   ```

   This Dockerfile defines a lightweight container running a Node.js environment, builds the Next.js app, and starts the application.

6. **Create a `docker-compose.yml` File:**
   In the root of your project, create a `docker-compose.yml` file to manage your Docker service:

   ```yaml
   version: '3.8'

   services:
     nextjs:
       build: .
       ports:
         - "3000:3000"
       environment:
         - NODE_ENV=production
       restart: always
   ```

   This file defines a single service for your Next.js application and exposes it on port 3000.

## Configuring Nginx

7. **Install Nginx:**
   If Nginx is not already installed on your server, install it using:

   ```sh
   sudo apt-get update
   sudo apt-get install nginx
   ```

8. **Create an Nginx Configuration File:**
   Create a new Nginx configuration file to set up Nginx as a reverse proxy for your Next.js application:

   ```sh
   sudo nano /etc/nginx/sites-available/yourdomain.conf
   ```

9. **Use the Following Configuration Template:**

   ```nginx
   # /etc/nginx/sites-available/yourdomain.conf

   server {
     listen 80;
     server_name your.domain;

     location / {
       proxy_pass http://localhost:3000;
       proxy_set_header Host $host;
       proxy_set_header X-Real-IP $remote_addr;
       proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
       proxy_set_header X-Forwarded-Proto $scheme;
     }

     # Optional: Serve static files directly
     location /_next/static/ {
       alias /var/www/yourdomain/_next/static/;
       expires 1y;
       access_log off;
     }

     # Additional configurations such as error pages, etc.
   }
   ```

   Replace `your.domain` with your actual domain. Ensure that the paths are updated according to your project’s directory structure.

10. **Enable the Nginx Configuration:**
    Create a symbolic link to enable the Nginx configuration file:

    ```sh
    sudo ln -s /etc/nginx/sites-available/yourdomain.conf /etc/nginx/sites-enabled/
    ```

11. **Test the Nginx Configuration:**
    Before restarting Nginx, test the configuration to ensure there are no syntax errors:

    ```sh
    sudo nginx -t
    ```

12. **Restart Nginx:**
    Apply the changes by restarting Nginx:

    ```sh
    sudo systemctl restart nginx
    ```

## Building and Running the Docker Container

13. **Build and Start the Docker Container:**
    Use Docker Compose to build and start the Next.js service:

    ```sh
    docker-compose up -d --build
    ```

    This command builds the Docker image and starts the container in detached mode.

14. **Verify the Deployment:**
    Once the container is up, you can check the logs to ensure everything is running smoothly:

    ```sh
    docker-compose logs -f
    ```

15. **Test the Application:**
    Open your browser and navigate to `http://your.domain` to verify that your Next.js application is running and accessible through Nginx.

## Managing the Deployment

16. **Restarting Services:**
    If you need to restart the Docker container, use the following command:

    ```sh
    docker-compose restart
    ```

17. **Stopping Services:**
    To stop the Docker container, run:

    ```sh
    docker-compose down
    ```

18. **Updating the Application:**
    When you make updates to your Next.js application, rebuild the Docker image and restart the service:

    ```sh
    docker-compose up -d --build
    ```

With these configurations, Docker, Nginx, and Next.js are connected and set up to work together seamlessly. Your Next.js project is now deployed and ready to handle production-level traffic.
