# Installing a Free SSL Certificate for Web Application on Nginx

This guide walks you through the process of securing your web application deployed on Nginx with a free SSL certificate, ensuring encrypted communication between clients and your server.

## Prerequisites

Before proceeding with the SSL certificate installation, make sure you have the following prerequisites:

- A subdomain associated with the IP address of your server.
- SSH access to your server with sudo privileges.
- Nginx installed and configured to serve your web application.
- A running application.

1. Create a subdomain and associate it with the IP address of your server. This can typically be done through your domain registrar's control panel or DNS management interface.

2. Install Certbot and Python-Certbot-Nginx, which are tools for obtaining SSL certificates from Let's Encrypt.
```sh
sudo apt update
sudo apt install certbot python3-certbot-nginx
```

3. Obtain the SSL Certificate by vun Certbot with the --nginx option to automatically configure SSL for Nginx:
```sh
sudo certbot --nginx
```

Follow the interactive prompts to select the domain (including the subdomain) you want to secure and provide an email address for renewal and security notices. Certbot will automatically update your Nginx configuration to enable HTTPS, handle the certificate issuance process with Let's Encrypt, and reload Nginx to apply the changes.

4. Once Certbot completes the installation process, verify the SSL configuration:
```sh
sudo nginx -t
```

If the test is successful, restart Nginx to apply the changes:

```sh
sudo systemctl restart nginx
```

5. Access your web application through the secured URL (https://your.subdomain) to ensure SSL is successfully configured and your application remains accessible.
