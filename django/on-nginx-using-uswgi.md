# How to deploy Django project on Nginx using uWSGI ?

Welcome to the deployment guide for deploying your Django project using uWSGI and Nginx. This comprehensive guide has been inspired by the informative article from [Tony Teaches Tech](https://tonyteaches.tech/django-nginx-uwsgi-tutorial/). We aim to provide you with step-by-step instructions to ensure a smooth and efficient deployment process, enabling your Django web application to handle production-level traffic with ease.

## Prerequisites

Before you begin, make sure you have the following:

- A working Django project.
- Python (version 3 or above) and pip installed on your server.
- Nginx installed on your server.
- Basic knowledge of Linux server administration.
- Access to a domain or subdomain for your Django application.

Now, let's dive into the deployment process and get your Django project up and running in a production environment!

## Setting Up the project

Follow these steps to prepare your Django project for deployment, setting up the virtual environment, installing requirements, configuring static files, and defining allowed hosts.

1. Create a virtual environment using `python -m venv env` or  `python3 -m venv env`
2. Activate the environment: `source env/bin/activate`
3. Install requirements: `pip install -r requiments.txt`
4. In your Django project's settings (`settings.py`), add your domain(s) to the `ALLOWED_HOSTS` list:

```python
# settings.py

ALLOWED_HOSTS = ['your_domain.com', 'www.your_domain.com']
```

5. Create folders named `media` and `static` at the root of your project. If your project involves media files (uploads, etc.) or static files (CSS, JS, etc.), these folders will be used to store them

6. Add the following configurations to your settings.py to manage static files:

```python
# settings.py

import os

# Other configurations
# ....

STATIC_URL = '/static/'
STATIC_ROOT = os.path.join(BASE_DIR, "static/")
```

7. Run the following command to collect static files into the static folder:

```sh
python manage.py collectstatic
```

## Setting Up uWSGI

8. Install uWSGI Library:

```sh
sudo apt-get update
sudo apt-get upgrade
sudo apt-get install gcc
pip install uwsgi
```

9. Create the `/path/to/your/django/project/uwsgi_params` file and fill it with the following content:

```sh
uwsgi_param  QUERY_STRING       $query_string;
uwsgi_param  REQUEST_METHOD     $request_method;
uwsgi_param  CONTENT_TYPE       $content_type;
uwsgi_param  CONTENT_LENGTH     $content_length;
uwsgi_param  REQUEST_URI        $request_uri;
uwsgi_param  PATH_INFO          $document_uri;
uwsgi_param  DOCUMENT_ROOT      $document_root;
uwsgi_param  SERVER_PROTOCOL    $server_protocol;
uwsgi_param  REQUEST_SCHEME     $scheme;
uwsgi_param  HTTPS              $https if_not_empty;
uwsgi_param  REMOTE_ADDR        $remote_addr;
uwsgi_param  REMOTE_PORT        $remote_port;
uwsgi_param  SERVER_PORT        $server_port;
uwsgi_param  SERVER_NAME        $server_name;
```

10. Create a `uwsgi.ini` file in the root directory of your project for uWSGI configuration. Use the following template:

```ini
[uwsgi]
# full path to Django project's root directory
chdir            = /path/to/your/django/project/

# Django's wsgi file
module           = uzaverse_manager.wsgi

# full path to Python virtual env
home             = /path/to/your/django/project/env

# enable uWSGI master process
master           = true

# maximum number of worker processes
processes        = 5

# the socket (use the full path to be safe)
socket           = /path/to/your/django/project/yourdomain.sock

# socket permissions
chmod-socket     = 666

# clear environment on exit
vacuum           = true

# daemonize uWSGI and write messages into the given log
daemonize        = /path/to/your/logs/folder/yourdomain.log
```

Make sure to customize the paths and configurations according to your project requirements. Note that you may need to create the specified log folder before running uWSGI.

## Configuring Nginx

11. Create a new Nginx configuration file

```sh
sudo nano /etc/nginx/sites-available/yourdomain.conf
```

12. Use the following template

```sh
# the upstream component nginx needs to connect to
upstream yourdomain_upstream {
  server unix:/path/to/your/django/project/yourdomain.sock;
}

# configuration of the server
server {
  listen      80;
  server_name your.domain;
  charset     utf-8;

  # max upload size
  client_max_body_size 24M;

  # Django media and static files
  location /media  {
    alias /path/to/your/django/project/media;
  }

  location /static {
    alias /path/to/your/django/project/static;
  }

  # Send all non-media requests to the Django server.
  location / {
    uwsgi_pass  yourdomain_upstream;
    include     /path/to/your/django/project/uwsgi_params;
  }
}
```
Replace `your.domain` with your actual domain and update the paths accordingly. If you have multiple domains, separate them with space.

13. Create a symbolic link to enable the Nginx configuration file:

```sh
sudo ln -s /etc/nginx/sites-available/yourdomain.conf /etc/nginx/sites-enabled/
```

14. Restart Nginx to apply the changes:

```sh
sudo systemctl restart nginx
```

## Connect Nginx, uWSGI and Django

15. Run uWSGI in emperor mode to monitor the uWSGI config file directory for changes and spawn vassals (instances) accordingly.

```sh
cd /path/to/your/django/project/env
mkdir vassals
sudo ln -s /path/to/your/django/project/uwsgi.ini /path/to/your/django/project/env/vassals/
```

16. Create a systemd service file at /etc/systemd/system/yourdomain.uwsgi.service with the following content:

```sh
[Unit]
Description=uwsgi emperor for yourdomain app
After=network.target
[Service]
User=<your user>
Restart=always
ExecStart=/path/to/your/django/project/env/bin/uwsgi --emperor /path/to/your/django/project/env/vassals --uid www-data --gid www-data
[Install]
WantedBy=multi-user.target
```
Replace `<your user>` with your actual username.

17. Enable the service to execute on system boot and start it:

```sh
systemctl enable yourdomain.uwsgi.service
systemctl start yourdomain.uwsgi.service
```

18. Check the status of the service and stop it if needed:

```sh
systemctl status yourdomain.uwsgi.service
systemctl stop yourdomain.uwsgi.service
```

With these configurations, Nginx, uWSGI, and Django are connected and set up to work together seamlessly. Your Django project is now deployed and ready to handle production-level traffic.
