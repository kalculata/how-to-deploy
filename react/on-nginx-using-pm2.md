# How to deploy React App on Nginx using pm2 ?

We aim to provide you with step-by-step instructions to ensure a smooth and efficient deployment process, enabling your React web application to handle production-level traffic with ease.

## Prerequisites

Before you start deploying your React app on Nginx using PM2, ensure you have the following prerequisites in place:

- A functional React app ready for deployment.
- Node.js and npm or Yarn installed on your server.
- Nginx installed on your server.
- Basic knowledge of Linux server administration.
- Access to a domain or subdomain for hosting your React application.

## Setting up project

1.  Build the production-ready version

```sh
yarn build # or npm run build
```

2. Install Express 

```sh
# With npm
npm install express

# Or with Yarn
yarn add express
```

3. Create a file named `server.js` at the root of the project and paste the following code:

```javascript
const express = require('express');
const path = require('path');

const app = express();
const port = process.env.PORT;

// Serve static files from the 'dist' directory (output directory after building the React app)
app.use(express.static(path.join(__dirname, 'dist')));

// Handle all routes by serving the 'index.html' file
app.get('*', (req, res) => {
  res.sendFile(path.join(__dirname, 'dist', 'index.html'));
});

// Start the server
app.listen(port, () => {
  console.log(`Server is running on port ${port}`);
});
```

This setup assumes that your React app is built using Vite (based on the 'dist' directory). If your build output directory is different, please update the paths accordingly in the `server.js` file. Additionally, you can modify the port variable to match your preferred port number.

Note:
The `server.js` file is particularly useful for ensuring the proper behavior of client-side routing in single-page applications (SPAs). In SPAs, when you refresh a page or directly access a route, traditional servers might return a 404 error since the server is not aware of client-side routes.

By using `server.js` with Express, as provided in this project setup, you address this issue. The server is configured to serve the 'index.html' file for all routes (*), allowing the client-side routing to take over. This helps prevent 404 errors when refreshing the page or accessing specific routes directly. The setup ensures a smooth navigation experience within your React application.

## Setting up PM2

PM2 is a process manager for Node.js applications. 

4. Install PM2 globally using npm or Yarn.

```sh
# With npm
npm install -g pm2

# Or with Yarn
yarn global add pm2
```

5. Create a `ecosystem.config.js` file in the root directory of your project for PM2 configuration with the following content:

```js
module.exports = {
  apps: [
    {
      name: '<project_name>',
      script: './server.js',
      instances: <instances>,
      watch: true,
      max_memory_restart: '<limit>M',
      env: {
        NODE_ENV: 'production',
        PORT: <port>
      },
      output: 'path/logs/out.log',   // Path to standard output log file
      error: 'path/logs/error.log',  // Path to error output log file
      log_date_format: 'DD-MM-YYYY HH:mm:ss',
    },
  ],
};
```

Make sure to replace the placeholders <project_name>, <instances>, <limit>, and <port> with the appropriate values for your deployment.

## Configuring Nginx

6. Create a new Nginx configuration file

```sh
sudo nano /etc/nginx/sites-available/project_name.conf
```

7. Use the following template

```sh
server {
  listen      80;
  server_name your.domain;
  charset     utf-8;

  location / {
    proxy_pass http://localhost:<port>;
    proxy_set_header Host $host;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection 'upgrade';
    proxy_cache_bypass $http_upgrade;
  }

  access_log off;
}
```
Replace `your.domain` with your actual domain and update the paths accordingly. If you have multiple domains, separate them with space.

8. Create a symbolic link to enable the Nginx configuration file:

```sh
sudo ln -s /etc/nginx/sites-available/project_name.conf /etc/nginx/sites-enabled/
```

9. Test the Nginx configuration and restart Nginx to apply the changes:

```sh
sudo nginx -t
sudo systemctl restart nginx
```

## Finial steps Nginx

10. To start the production version of your app using PM2

```sh
pm2 start ecosystem.config.js
```

10. Confirm that your app is running and monitored by PM2. You can check the logs using:

```sh
pm2 logs '<project_name>'
pm2 status 
```
