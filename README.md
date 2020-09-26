# Deploy Django on Debian server

1. Connect to server via root and create new user
2. Setup SSH
3. Install must have packages
4. Install python
5. Clone the project
6. Install and configure PostgreSQL
7. Install packages from requirements.txt
8. Configure nginx, gunicorn, supervisor
9. Configre static and media
10. (optional) SSL for domain

Simplifications:
1. Remote server host: example.com
2. Remote user: a1
3. Core directory (inside home): backend (/home/a1/backend)
4. Virtualenv directory: /home/a1/backend/ienv/
5. Main project directory: /home/a1/backend/project/

-----
Step 1. Connect to server through SSH. Then
```
sudo adduser a1
sudo usermod -aG sudo a1
```

Local machine:
```
ssh-copy-id a1@example.com
sudo mcedit /ssh/ssh_config
Host example
HostName example.com
User a1
Port 2022
```

Step 2. Connect to server via key and then:
```
sudo mcedit /etc/ssh/sshd_config
PermitRootLogin no
PasswordAuthentication no
Port 2022
sudo service ssh restart
```
Now we may connect to server via `ssh example` without password.

Step 3. Install packages.
```
sudo apt-get install -y zsh tree redis-server nginx libssl-dev zlib1g-dev libbz2-dev libreadline-dev libsqlite3-dev llvm libncurses5-dev libncursesw5-dev xz-utils tk-dev libffi-dev liblzma-dev python3-dev python3-lxml libxslt-dev python-libxml2 python-libxslt1 libffi-dev libssl-dev python-dev gnumeric libsqlite3-dev libpq-dev libxml2-dev libxslt1-dev libjpeg-dev libfreetype6-dev libcurl4-openssl-dev supervisor gcc python3-setuptools
```
```
oh-my-zsh:
sh -c "$(curl -fsSL https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"
```
Step 4. Install python from source.
```
wget https://www.python.org/ftp/python/3.7.9/Python-3.7.9.tgz
tar xvf Python-3.7.9
cd Python-3.7.9
mkdir ~/.python
./configure --enable-optimizations --prefix=/home/a1/.python
make -j8
sudo make altinstall
```

Add python to path:
```
sudo mcedit ~/.zshrc
export PATH=$PATH:/home/a1/.python/bin
```

Step 5. Cloning project from repo.

Step 6. Install and configure postgreSQL
```
Prepare:
wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -
echo "deb http://apt.postgresql.org/pub/repos/apt/ `lsb_release -cs`-pgdg main" |sudo tee  /etc/apt/sources.list.d/pgdg.list
sudo apt-get update

Installation:
sudo apt-get install postgresql-12 postgresql-client-12

Create user for project:
sudo -u postgres psql
create user a1dbuser with password 'password';
create database a1db with owner a1dbuser;
\q;
```

Step 7. Install packages
```
Create virtual environment:
python3.7 -m venv ienv
source ienv/bin/activate

Finally:
pip install -r requirements.txt

```

Step 8. Nginx, Gunicorn, Supervisor
Nginx:
file path: /etc/nginx/sites-enabled/site.conf

```
upstream main {
  server unix:///var/tmp/main.sock;
}

server {
    listen 80;
    server_name {domain_or_ip};
    access_log    /var/log/nginx/access.log;
    error_log    /var/log/nginx/error.log;

    location /static/ {
        alias    /var/www/static/;
    }

    location /media/ {
        alias    /var/www/media/;
    }
    
    location ~ /.well-known {
        allow all;
    }

    location / {
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header Host $http_host;
        proxy_redirect off;

        if (!-f $request_filename) {
            proxy_pass http://main;
            break;
        }
    }
}

sudo ln -s /etc/nginx/site-available/site.conf /etc/nginx/sites-enabled/site.conf
sudo service nginx restart
```

Gunicorn: 
file path: /home/a1/backend/bin/gunicorn_script.bash
```
#!/bin/bash
NAME="gunicorn_app"                                 
DJANGODIR="/home/a1/backend/"      
SOCKFILE="/var/tmp/main.sock"
USER="www-data"                                
GROUP="www-data"                     
NUM_WORKERS=3                           
DJANGO_SETTINGS_MODULE="project.settings"           
DJANGO_WSGI_MODULE="project.wsgi" 

cd $DJANGODIR
source ienv/bin/activate
export DJANGO_SETTINGS_MODULE=$DJANGO_SETTINGS_MODULE
export PYTHONPATH=$DJANGODIR:$PYTHONPATH

RUNDIR=$(dirname $SOCKFILE)
test -d $RUNDIR || mkdir -p $RUNDIR

exec ienv/bin/gunicorn ${DJANGO_WSGI_MODULE}:application \
  --name $NAME \
  --workers $NUM_WORKERS \
  --threads=2\
  --user=$USER \
  --group=$GROUP \
  --bind=unix:$SOCKFILE \
  --log-level=debug \
  --log-file=-
```

Supervisor:
file path: /etc/supervisor/conf/main.conf

```
[program:core-gunicorn]
command = /home/a1/backend/bin/gunicorn_script.bash
user = www-data                                                         
stdout_logfile = /home/a1/backend/logs/gunicorn.log  
redirect_stderr = true                                    
environment=LANG=en_US.UTF-8,LC_ALL=en_US.UTF-8

sudo supervisorctl reread
sudo supervisorctl update
sudo supervisorctl restart all
```

Step 9. Media and Static.
I have many problems with static and media files on production.
For example, 403 Forbidden error.
Finally, I found solution:

```
sudo groupadd varwwwusers
sudo adduser www-data a1 varwwwusers
sudo chgrp -R varwwwusers /var/www/
sudo chmod 770 -R /var/www/
sudo chmod g+s  /var/www/
```
Now we don't have any problems with permissions with media/static/.

Go to the project directory and:
```
python manage.py collectstatic
```



