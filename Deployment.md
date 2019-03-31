# Django Deployment on Ubuntu

Xuewei Jia, 2019.03.25

Relations:  
`sitename.conf` --> `wsgi.py` --> `settings.py` --> `urls.py` --> `views.py`

## Preparation

### Install `Apache` and `mod_wsgi` package

```sh
# Apache install
sudo apt install apache2
# Without apache-dev, mod_wsgi won't be installed for missing apax, an extensive tool for Apache
sudo apt install apache2-dev

# mod_wsgi install
pip install mod_wsgi
sudo apt-get install libapache2-mod-wsgi-py3

# Test mod_wsgi, running on localhost:8000
mod_wsgi-express start-server
```

## Deploy on `virtualenv`

### Create a `virtualenv`

```sh
# Create a virtualenv with specific version of python intepreter
virtualenv --python=python3.5 my_project

# Activate virtualenv
# Command source: Execute commands from a file in the current shell.
source my_project/bin/activate

# Deactivate
deactivate
```

### Configure `Apache` with `mod_wsgi`

#### Setting Global `WSGIPythonHome`

View [Official Documentation](https://docs.djangoproject.com/en/2.1/howto/deployment/wsgi/modwsgi/)

```sh
# /etc/apache2/apache2.conf, add
# WSGIPythonHome /path/to/virtualenv
# WSGIPythonPath /path/to/project_root/in/virtualenv
WSGIPythonHome /home/ubuntu/virtualenvTest/my_project
WSGIPythonPath /home/ubuntu/virtualenvTest/my_project/VirginInDocker

# Fixing UnicodeEncodeError for file uploads
# /etc/apache2/envvars
export LANG='en_US.UTF-8'
export LC_ALL='en_US.UTF-8'
```

#### Configure Virtual Host

[Virtual Environment and Python Version in `mod_wsgi`](https://modwsgi.readthedocs.io/en/develop/user-guides/virtual-environments.html#virtual-environment-and-python-version)

[Daemon Mode](https://modwsgi.readthedocs.io/en/develop/user-guides/virtual-environments.html#daemon-mode-single-application)

```sh
# /etc/apache2/sites-available, create a new file *.conf
# cp 000-default.conf virgin.conf
<VirtualHost *:80>
    ServerName www.xueweijia.cn
    ServerAlias xueweijia.cn
    ServerAdmin asjxw@foxmail.com

    # Serving static and media files
    # Alias /media/ /path/to/media/directory
    Alias /media/ /home/ubuntu/virtualenvTest/my_project/VirginInDocker/media/
    # Alias /static/ /path/to/static/directory
    Alias /static/ /home/ubuntu/virtualenvTest/my_project/VirginInDocker/static/

    ## Ensure media and static can be found
    # <Directory /path/to/media/directory>
    <Directory /home/ubuntu/virtualenvTest/my_project/VirginInDocker/media>
    Require all granted
    </Directory>
    # <Directory /path/to/static/directory>
    <Directory /home/ubuntu/virtualenvTest/my_project/VirginInDocker/static>
    Require all granted
    </Directory>

    # WSGI configuration
    # Resolve wsgi.py to root dir(~/)
    # WSGIScriptAlias / /path/to/project
    WSGIScriptAlias / /home/ubuntu/virtualenvTest/my_project/VirginInDocker/Virgin/wsgi.py
    # Using mod_wsgi daemon mode
    # WSGIDaemonProcess example.com python-home=/path/to/venv python-path=/path/to/project
    # WSGIDaemonProcess directive defines the daemon process group
    WSGIDaemonProcess xueweijia.com python-home=/home/ubuntu/virtualenvTest/my_project python-path=/home/ubuntu/virtualenvTest/my_project/VirginInDocker
    # WSGIProcessGroup exanple.com
    # WSGIProcessGroup directive indicates that the WSGI application should be run within the defined daemon process group.
    WSGIProcessGroup xueweijia.cn
</VirtualHost>
```

#### Edit `Django` `ALLOWED_HOSTS`

```python
# Settings.py
ALLOWED_HOSTS = ['xueweijia.cn', 'www.xueweijia.cn']
```

### Configure `Redis`

#### Install `Redis`

```sh
sudo apt install redis-server
```

#### Configure `Redis` hostname and port

##### Change system `Redis` configuration

```sh
# /etc/redis/redis.conf
bind 127.0.0.1
```

##### Change `django`'s package `django_redis` default configuration

```python
# /django_project/settings.py
CACHES = {
    "default": {
        "BACKEND": "django_redis.cache.RedisCache",
        # Change Redis hostname and port here
        "LOCATION": "redis://127.0.0.1:6379/1",
        "OPTIONS": {
            "CLIENT_CLASS": "django_redis.client.DefaultClient",
            "PASSWORD": "mysecret"
        }
    }
}
```

#### Test and usage

```sh
# In Ubuntu
redis-cli set 'hello' 'thankyou'
redis-cli get 'hello'

# In Django
python manage.py shell
from django_redis import get_redis_connection
redis_connection = get_redis_connection('default')
redis_connection.set('hello', 'thankyou')
redis_connection.get('hello')
```

### File Authority

```sh
sudo chgrp www-data VirginInDocker
sudo chmod g+w VirginInDocker
sudo chgrp www-data VirginInDocker/db.sqlite3
sudo chmod g+w VirginInDocker/db.sqlite3
```

### Runserver

```sh
sudo systemctl start apache2.service
# Reload
sudo systemctl reload apache2.service
```

## Backup

* Local scheduled backup
* Remote mirror backup

Local server:

```sh
# backup.shell
cp /path/to/sqlite3 /path/to/target/backup/$(date +%m%d%H%M)_db.sqlite3
find /path/to/packup/dir -mtime +3 -name "sqlite3" -exec rm {} \;
# terminal
crontab -e
# crontab
0 */4 0 0 0 sh /path/to/backup.shell
```

Remote mirror:

```sh
# mirror.shell
# -e execute the command just after selecting
# mirror -e delete local if remote file does not exists
# mirror -n mirror newer file only
lftp username:passwd@hostname:port -e "mirror -e -n /remote/ /local/"
# terminal
crontab -e
# crontab
0 0 * * * sh /path/to/mirror.shell
```