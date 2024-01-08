# How to deploy Node.js App on Nginx using pm2 ?

We aim to provide you with step-by-step instructions to ensure a smooth and efficient deployment process, enabling your Node.js web application to handle production-level traffic with ease.

## Prerequisites

Before you start deploying your Node.js app on Nginx using PM2, ensure you have the following prerequisites in place:

- A functional Node.js app ready for deployment.
- Node.js and npm or Yarn installed on your server.
- Nginx installed on your server.
- Basic knowledge of Linux server administration.
- Access to a domain or subdomain for hosting your Node.js application.

## Setting up project

1.  Install the required packages:

```sh
yarn install # or npm i
```

## Setting up PM2

PM2 is a process manager for Node.js applications.

2. Install PM2 globally using npm or Yarn.

```sh
# With npm
npm install -g pm2

# Or with Yarn
yarn global add pm2
```

3. Create a `ecosystem.config.js` file in the root directory of your project for PM2 configuration with the following content:

```js
module.exports = {
  apps: [
    {
      name: '<project_name>',
      script: 'path/to/your/app.js',
      instances: <instances>,
      watch: true,
      max_memory_restart: '<limit>M',
      env: {
        NODE_ENV: 'production',
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

4. Create a new Nginx configuration file

```sh
sudo nano /etc/nginx/sites-available/project_name.conf
```

5. Use the following template

```sh
server {
  listen      80;
  server_name your.domain;
  charset     utf-8;

  location / {
      proxy_pass http://localhost:<port>;
      proxy_http_version 1.1;
      proxy_set_header Upgrade $http_upgrade;
      proxy_set_header Connection 'upgrade';
      proxy_set_header Host $host;
      proxy_cache_bypass $http_upgrade;
  }

  access_log off;
}
```
Replace `your.domain` with your actual domain and update the paths accordingly. If you have multiple domains, separate them with space.

6. Create a symbolic link to enable the Nginx configuration file:

```sh
sudo ln -s /etc/nginx/sites-available/project_name.conf /etc/nginx/sites-enabled/
```

7. Test the Nginx configuration and restart Nginx to apply the changes:

```sh
sudo nginx -t
sudo systemctl restart nginx
```

## Finial steps Nginx

8. To start the production version of your app using PM2

```sh
pm2 start ecosystem.config.js
```

9. Confirm that your app is running and monitored by PM2. You can check the status using:

```sh
pm2 status
```
