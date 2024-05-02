# How to Deploy Java Web App on Nginx Using Tomcat9 ?

This guide provides step-by-step instructions to deploy a Java web application on Nginx using Tomcat9, ensuring reliable process management and efficient handling of production-level traffic.

## Prerequisites

Before deploying your Java web application on Nginx using Tomcat9, ensure you have the following prerequisites in place:

- A functional Java web application ready for deployment.
- Basic knowledge of Linux server administration.
- Nginx installed on your server.
- Tomcat9 installed on your server.
- Access to a domain or subdomain for hosting your application.

## Setting up the Project

1. Prepare your Java web application for deployment.

2. Ensure your application is configured to run on Tomcat9.

## Configuring Tomcat9

Tomcat9 is an open-source implementation of the Java Servlet, JavaServer Pages, Java Expression Language, and Java WebSocket technologies.

3. Prepare your WAR file: Make sure your WAR file is ready for deployment. This typically involves building your web application and packaging it into a WAR file using tools like Apache Maven or Gradle.

4. Copy the WAR file to the Tomcat webapps directory: Once your WAR file is ready, you'll need to copy it to the `"webapps"` directory within your Tomcat installation. You can use `SCP` or `SFTP` to transfer the WAR file from your local machine to the server. The webapps directory is usually located within the Tomcat installation directory, for example: `/opt/tomcat/webapps`.

5. (Only if you want to change the default Tomcat port) Configure Tomcat to run on a specific port:
  - Open the `server.xml` file located in the conf directory of your Tomcat installation (e.g., `/opt/tomcat/conf/server.xml`).
  - Find the `<Connector>` element with the `protocol="HTTP/1.1"` attribute. This is the default HTTP connector for Tomcat.
  - Change the port attribute to the desired port number (e.g., 8081).
  - Save the file.

6. Restart Tomcat for the changes to take effect

```sh
sudo systemctl restart tomcat9
```

## Configuring Nginx

4. Create a new Nginx configuration file for your Java web application:

```sh
sudo nano /etc/nginx/sites-available/myjavaapp.conf
```

5. Use the following template for your Nginx configuration:

```sh
server {
  listen 80;
  server_name your.domain;

  location / {
    proxy_pass http://localhost:<port>/myapp; # Assuming your Java web app is deployed under the context path /myapp
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;
  }

  access_log off;
}

```
Remember to replace your.domain with your actual domain and `<port>` with the port number where Tomcat is running.

6. Create a symbolic link to enable the Nginx configuration file:

```sh
sudo ln -s /etc/nginx/sites-available/myjavaapp.conf /etc/nginx/sites-enabled/
```

7. Test the Nginx configuration and restart Nginx to apply the changes:

```sh
sudo nginx -t
sudo systemctl restart nginx
```

## Final Steps

8. Access your Java web application through your domain to ensure it is successfully deployed and accessible.