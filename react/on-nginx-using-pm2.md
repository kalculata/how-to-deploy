# How to deploy React App on Nginx using pm2 ?

We aim to provide you with step-by-step instructions to ensure a smooth and efficient deployment process, enabling your React web application to handle production-level traffic with ease.

## Prerequisites

Before you start deploying your React app on Nginx using PM2, ensure you have the following prerequisites in place:

- A functional React app ready for deployment.
- Node.js and npm or Yarn installed on your server.
- Nginx installed on your server.
- Basic knowledge of Linux server administration.
- Access to a domain or subdomain for hosting your React application.

## Setting up PM2

PM2 is a process manager for Node.js applications. 

1. Install PM2 globally using npm or Yarn.

```sh
# With npm
npm install -g pm2

# Or with Yarn
yarn global add pm2
```

2. Create a `ecosystem.config.js` file in the root directory of your project for PM2 configuration with the following content:

```js
module.exports = {
  apps: [
    {
      name: '<project_name>',
      script: 'yarn',
      args: 'start',
      instances: <instances>,
      watch: true,
      max_memory_restart: '<limit>M',
      env: {
        PORT: <port>,
        NODE_ENV: 'production',
      },
    },
  ],
};
```

Make sure to replace the placeholders <project_name>, <instances>, <limit>, and <port> with the appropriate values for your deployment.
