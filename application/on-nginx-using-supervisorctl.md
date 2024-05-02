# How to Deploy Application on Nginx Using Supervisorctl

This guide provides step-by-step instructions to deploy an application on Nginx using Supervisorctl, ensuring reliable process management and efficient handling of production-level traffic. While this example uses a Java JAR application, the process remains the same for applications built with different technologies.

## Prerequisites

Before deploying your application on Nginx using Supervisorctl, ensure you have the following prerequisites in place:

- A functional application ready for deployment.
- Basic knowledge of Linux server administration.
- Nginx installed on your server.
- Supervisor installed on your server.
- Access to a domain or subdomain for hosting your application.

## Setting up the Project

1. Prepare your application for deployment.

2. Ensure your application is configured to run as a service or daemon.

## Configuring Supervisorctl

Supervisorctl is a client/server system that allows its users to monitor and control a number of processes on UNIX-like operating systems.

3. Create a Supervisor configuration file for your application. For example, create a file named `myapp.conf` in the Supervisor configuration directory (commonly `/etc/supervisor/conf.d/`) with the following content:

```ini
[program:myapp]
command=/path/to/java -jar -Dserver.port=<port> /path/to/your/application.jar
directory=/path/to/your/application
user=<your_user>
autostart=true
autorestart=true
stderr_logfile=/var/log/myapp.err.log
stdout_logfile=/var/log/myapp.out.log
```
Ensure you substitute `<port>` with the designated port number for your Java application's listening configuration. This guarantees the JAR executes with the specified port settings. Also, replace `/path/to/java`, `/path/to/your/application.jar`, `/path/to/your/application`, and `<your_user>` with the appropriate values for your deployment.

4. Update Supervisor to read the new configuration and start your application:

```sh
sudo supervisorctl reread
sudo supervisorctl update
sudo supervisorctl start myapp
```

## Configuring Nginx

5. Create a new Nginx configuration file for your application:

```sh
sudo nano /etc/nginx/sites-available/myapp.conf
```

6. Use the following template for your Nginx configuration:

```sh
server {
  listen 80;
  server_name your.domain;

  location / {
    proxy_pass http://localhost:<port>;
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;
  }

  access_log off;
}
```

Replace your.domain with your actual domain and update <port> accordingly.

7. Create a symbolic link to enable the Nginx configuration file:

```sh
sudo ln -s /etc/nginx/sites-available/myapp.conf /etc/nginx/sites-enabled/
```

8. Test the Nginx configuration and restart Nginx to apply the changes:

```sh
Copy code
sudo nginx -t
sudo systemctl restart nginx
```

## Final Steps

9. Confirm that your application is running and monitored by Supervisor. You can check the status using:

```sh
sudo supervisorctl status
```

10. Access your application through your domain to ensure it is successfully deployed and accessible.