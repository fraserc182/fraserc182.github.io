---
layout: post
title: Installing Netbox on CentOS 7.8
date: 2021-03-24 12:45
author: Fraser Clark
comments: true
tags: [netbox]
---
<!-- wp:paragraph -->
<p>I've found the official docs to be lacking when installing netbox on CentOS.<br>When moving Netbox to a new server I scripted the entire install, this should be easy enough to modify if required or just copy the steps and do it manually.</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p>When using the script, the only extra part that is needed is the below configuration file which has specific values which the sed command then finds and replaces.</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p>When using the script, read through it and update the required variables as there as some placeholders in there.<br>On line 78, ensure you update where you are copying the file from.<br>On line 114, update the certificate details. Feel free to modify this and use your own certificates.</p>
<!-- /wp:paragraph -->

<!-- wp:code -->
<pre class="wp-block-code"><code>
{% highlight bash %}
#########################
#                       #
#   Required settings   #
#                       #
#########################

# This is a list of valid fully-qualified domain names (FQDNs) for the NetBox server. NetBox will not permit write
# access to the server via any other hostnames. The first FQDN in the list will be treated as the preferred name.
#
# Example: ALLOWED_HOSTS = &#091;'netbox.example.com', 'netbox.internal.local']
ALLOWED_HOSTS = &#091;'127.0.0.1', 'var_ipaddress', 'var_hostname'] ## variables required here

# PostgreSQL database configuration. See the Django documentation for a complete list of available parameters:
#   https://docs.djangoproject.com/en/stable/ref/settings/#databases
DATABASE = {
    'NAME': 'netbox',         # Database name
    'USER': 'netbox',               # PostgreSQL username
    'PASSWORD': 'var_dbpassword',           # PostgreSQL password
    'HOST': 'localhost',      # Database server
    'PORT': '',               # Database port (leave blank for default)
    'CONN_MAX_AGE': 300,      # Max database connection age
}

# Redis database settings. Redis is used for caching and for queuing background tasks such as webhook events. A separate
# configuration exists for each. Full connection details are required in both sections, and it is strongly recommended
# to use two separate database IDs.
REDIS = {
    'tasks': {
        'HOST': 'localhost',
        'PORT': 6379,
        # Comment out `HOST` and `PORT` lines and uncomment the following if using Redis Sentinel
        # 'SENTINELS': &#091;('mysentinel.redis.example.com', 6379)],
        # 'SENTINEL_SERVICE': 'netbox',
        'PASSWORD': '',
        'DATABASE': 0,
        'DEFAULT_TIMEOUT': 300,
        'SSL': False,
    },
    'caching': {
        'HOST': 'localhost',
        'PORT': 6379,
        # Comment out `HOST` and `PORT` lines and uncomment the following if using Redis Sentinel
        # 'SENTINELS': &#091;('mysentinel.redis.example.com', 6379)],
        # 'SENTINEL_SERVICE': 'netbox',
        'PASSWORD': '',
        'DATABASE': 1,
        'DEFAULT_TIMEOUT': 300,
        'SSL': False,
    }
}

# This key is used for secure generation of random numbers and strings. It must never be exposed outside of this file.
# For optimal security, SECRET_KEY should be at least 50 characters in length and contain a mix of letters, numbers, and
# symbols. NetBox will not run without this defined. For more information, see
# https://docs.djangoproject.com/en/stable/ref/settings/#std:setting-SECRET_KEY
SECRET_KEY = 'var_secretkey'


#########################
#                       #
#   Optional settings   #
#                       #
#########################

# Specify one or more name and email address tuples representing NetBox administrators. These people will be notified of
# application errors (assuming correct email settings are provided).
ADMINS = &#091;
     &#091;'admin', 'admin@admin.com']

]

# URL schemes that are allowed within links in NetBox
ALLOWED_URL_SCHEMES = (
    'file', 'ftp', 'ftps', 'http', 'https', 'irc', 'mailto', 'sftp', 'ssh', 'tel', 'telnet', 'tftp', 'vnc', 'xmpp',
)

# Optionally display a persistent banner at the top and/or bottom of every page. HTML is allowed. To display the same
# content in both banners, define BANNER_TOP and set BANNER_BOTTOM = BANNER_TOP.
BANNER_TOP = ''
BANNER_BOTTOM = ''

# Text to include on the login page above the login form. HTML is allowed.
BANNER_LOGIN = ''

# Base URL path if accessing NetBox within a directory. For example, if installed at http://example.com/netbox/, set:
# BASE_PATH = 'netbox/'
BASE_PATH = ''

# Cache timeout in seconds. Set to 0 to dissable caching. Defaults to 900 (15 minutes)
CACHE_TIMEOUT = 900

# Maximum number of days to retain logged changes. Set to 0 to retain changes indefinitely. (Default: 90)
CHANGELOG_RETENTION = 90

# API Cross-Origin Resource Sharing (CORS) settings. If CORS_ORIGIN_ALLOW_ALL is set to True, all origins will be
# allowed. Otherwise, define a list of allowed origins using either CORS_ORIGIN_WHITELIST or
# CORS_ORIGIN_REGEX_WHITELIST. For more information, see https://github.com/ottoyiu/django-cors-headers
CORS_ORIGIN_ALLOW_ALL = False
CORS_ORIGIN_WHITELIST = &#091;
    # 'https://hostname.example.com',
]
CORS_ORIGIN_REGEX_WHITELIST = &#091;
    # r'^(https?://)?(\w+\.)?example\.com$',
]

# Set to True to enable server debugging. WARNING: Debugging introduces a substantial performance penalty and may reveal
# sensitive information about your installation. Only enable debugging while performing testing. Never enable debugging
# on a production system.
DEBUG = False

# Email settings
EMAIL = {
    'SERVER': 'localhost',
    'PORT': 25,
    'USERNAME': '',
    'PASSWORD': '',
    'USE_SSL': False,
    'USE_TLS': False,
    'TIMEOUT': 10,  # seconds
    'FROM_EMAIL': '',
}

# Enforcement of unique IP space can be toggled on a per-VRF basis. To enforce unique IP space within the global table
# (all prefixes and IP addresses not assigned to a VRF), set ENFORCE_GLOBAL_UNIQUE to True.
ENFORCE_GLOBAL_UNIQUE = False

# Exempt certain models from the enforcement of view permissions. Models listed here will be viewable by all users and
# by anonymous users. List models in the form `&lt;app&gt;.&lt;model&gt;`. Add '*' to this list to exempt all models.
EXEMPT_VIEW_PERMISSIONS = &#091;
    # 'dcim.site',
    # 'dcim.region',
    # 'ipam.prefix',
]

# HTTP proxies NetBox should use when sending outbound HTTP requests (e.g. for webhooks).
# HTTP_PROXIES = {
#     'http': 'http://10.10.1.10:3128',
#     'https': 'http://10.10.1.10:1080',
# }

# IP addresses recognized as internal to the system. The debugging toolbar will be available only to clients accessing
# NetBox from an internal IP.
INTERNAL_IPS = ('127.0.0.1', '::1')

# Enable custom logging. Please see the Django documentation for detailed guidance on configuring custom logs:
#   https://docs.djangoproject.com/en/stable/topics/logging/
LOGGING = {}
# Setting this to True will permit only authenticated users to access any part of NetBox. By default, anonymous users
# are permitted to access most data in NetBox (excluding secrets) but not make any changes.
LOGIN_REQUIRED = False

# The length of time (in seconds) for which a user will remain logged into the web UI before being prompted to
# re-authenticate. (Default: 1209600 &#091;14 days])
LOGIN_TIMEOUT = None

# Setting this to True will display a "maintenance mode" banner at the top of every page.
MAINTENANCE_MODE = False

# An API consumer can request an arbitrary number of objects =by appending the "limit" parameter to the URL (e.g.
# "?limit=1000"). This setting defines the maximum limit. Setting it to 0 or None will allow an API consumer to request
# all objects by specifying "?limit=0".
MAX_PAGE_SIZE = 1000

# The file path where uploaded media such as image attachments are stored. A trailing slash is not needed. Note that
# the default value of this setting is derived from the installed location.
# MEDIA_ROOT = '/opt/netbox/netbox/media'

# By default uploaded media is stored on the local filesystem. Using Django-storages is also supported. Provide the
# class path of the storage driver in STORAGE_BACKEND and any configuration options in STORAGE_CONFIG. For example:
# STORAGE_BACKEND = 'storages.backends.s3boto3.S3Boto3Storage'
# STORAGE_CONFIG = {
#     'AWS_ACCESS_KEY_ID': 'Key ID',
#     'AWS_SECRET_ACCESS_KEY': 'Secret',
#     'AWS_STORAGE_BUCKET_NAME': 'netbox',
#     'AWS_S3_REGION_NAME': 'eu-west-1',
# }

# Expose Prometheus monitoring metrics at the HTTP endpoint '/metrics'
METRICS_ENABLED = False

# Credentials that NetBox will uses to authenticate to devices when connecting via NAPALM.
NAPALM_USERNAME = ''
NAPALM_PASSWORD = ''

# NAPALM timeout (in seconds). (Default: 30)
NAPALM_TIMEOUT = 30

# NAPALM optional arguments (see http://napalm.readthedocs.io/en/latest/support/#optional-arguments). Arguments must
# be provided as a dictionary.
NAPALM_ARGS = {}

# Determine how many objects to display per page within a list. (Default: 50)
PAGINATE_COUNT = 50

# Enable installed plugins. Add the name of each plugin to the list.
PLUGINS = &#091;]

# Plugins configuration settings. These settings are used by various plugins that the user may have installed.
# Each key in the dictionary is the name of an installed plugin and its value is a dictionary of settings.
# PLUGINS_CONFIG = {
#     'my_plugin': {
#         'foo': 'bar',
#         'buzz': 'bazz'
#     }
# }

# When determining the primary IP address for a device, IPv6 is preferred over IPv4 by default. Set this to True to
# prefer IPv4 instead.
PREFER_IPV4 = False

# Rack elevation size defaults, in pixels. For best results, the ratio of width to height should be roughly 10:1.
RACK_ELEVATION_DEFAULT_UNIT_HEIGHT = 22
RACK_ELEVATION_DEFAULT_UNIT_WIDTH = 220

# Remote authentication support
REMOTE_AUTH_ENABLED = False
REMOTE_AUTH_BACKEND = 'netbox.authentication.RemoteUserBackend'
REMOTE_AUTH_HEADER = 'HTTP_REMOTE_USER'
REMOTE_AUTH_AUTO_CREATE_USER = True
REMOTE_AUTH_DEFAULT_GROUPS = &#091;]
REMOTE_AUTH_DEFAULT_PERMISSIONS = &#091;]

# This determines how often the GitHub API is called to check the latest release of NetBox. Must be at least 1 hour.
RELEASE_CHECK_TIMEOUT = 24 * 3600

# This repository is used to check whether there is a new release of NetBox available. Set to None to disable the
# version check or use the URL below to check for release in the official NetBox repository.
RELEASE_CHECK_URL = 'https://api.github.com/repos/netbox-community/netbox/releases'

# The file path where custom reports will be stored. A trailing slash is not needed. Note that the default value of
# this setting is derived from the installed location.
# REPORTS_ROOT = '/opt/netbox/netbox/reports'

# The file path where custom scripts will be stored. A trailing slash is not needed. Note that the default value of
# this setting is derived from the installed location.
# SCRIPTS_ROOT = '/opt/netbox/netbox/scripts'

# By default, NetBox will store session data in the database. Alternatively, a file path can be specified here to use
# local file storage instead. (This can be useful for enabling authentication on a standby instance with read-only
# database access.) Note that the user as which NetBox runs must have read and write permissions to this path.
SESSION_FILE_PATH = None

# Time zone (default: UTC)
TIME_ZONE = 'UTC'

# Date/time formatting. See the following link for supported formats:
# https://docs.djangoproject.com/en/stable/ref/templates/builtins/#date
DATE_FORMAT = 'N j, Y'
SHORT_DATE_FORMAT = 'Y-m-d'
TIME_FORMAT = 'g:i a'
SHORT_TIME_FORMAT = 'H:i:s'
DATETIME_FORMAT = 'N j, Y g:i a'
SHORT_DATETIME_FORMAT = 'Y-m-d H:i'</code></pre>
<!-- /wp:code -->

<!-- wp:code -->
<pre class="wp-block-code"><code>#!/bin/bash

# send everything to /var/log/netboxinstall.log
exec 3&gt;&amp;1 4&gt;&amp;2
trap 'exec 2&gt;&amp;4 1&gt;&amp;3' 0 1 2 3
exec 1&gt;/var/log/netboxinstall.log 2&gt;&amp;1

# variables

var_ipaddress=`ip a |grep eth|grep inet|awk '{print $2}'|sed 's/\/.*//'`
var_hostname=$(hostname | tr '&#091;:upper:]' '&#091;:lower:]' | cut -c 4-)
var_hostname+=".MYDOMAIN.COM"
var_dbpassword=`cat /dev/urandom | tr -dc '&#091;:alnum:]' | head -c 16`
var_secretkey=`cat /dev/urandom | tr -dc '&#091;:alnum:]' | head -c 50`
version="2.10.6"

## install system packages
echo Installing packages
yum install -y gcc python36 python36-devel python3-pip libxml2-devel libxslt-devel libffi-devel openssl-devel redhat-rpm-config
## upgrade pip
pip3 install --upgrade pip

## install postgres &amp; configure
## postgres 9.2 is availabed through yum in centos 7
## netbox requires 9.6 or greater. For this reason I am installing version 13 which is currently the latest stable version

echo installing and configuring postgres
yum install -y https://download.postgresql.org/pub/repos/yum/reporpms/EL-7-x86_64/pgdg-redhat-repo-latest.noarch.rpm
yum install -y postgresql13-server
postgresql-13-setup initdb
systemctl enable postgresql-13
systemctl start postgresql-13
## find and replace "ident" with "md5" in postgres conf file
## by default centos configures ident authentication, we require md5

sed -i 's/ident/md5/g' /var/lib/pgsql/13/data/pg_hba.conf
## run commands to initialise database
echo creating database and user
sudo -u postgres psql -c "CREATE DATABASE netbox;"
sudo -u postgres psql -c "CREATE USER netbox WITH PASSWORD '$var_dbpassword';"
sudo -u postgres psql -c "GRANT ALL PRIVILEGES ON DATABASE netbox TO netbox;"
## restarting because we have edited the pg_hba.conf file
systemctl restart postgresql-13.service

## install redis &amp; configure
## for netbox we require redis v4 or greater. for this reason I am installing version 6

echo installing redis
yum install -y http://rpms.remirepo.net/enterprise/remi-release-7.rpm
yum --enablerepo=remi install -y redis
rpm -qi redis
systemctl enable --now redis
systemctl restart redis
systemctl status redis

## test redis is running, we should see PONG if successful
redis-cli ping

## download &amp; install netbox
echo downloading and installing netbox

wget https://github.com/netbox-community/netbox/archive/v$version.tar.gz
tar -xzf v$version.tar.gz -C /opt
ln -s /opt/netbox-$version/ /opt/netbox

## create netbox user and adjust permissions of netbox folders
echo creating netbox user and modifying permissions
groupadd --system netbox
adduser --system -g netbox netbox
chown --recursive netbox /opt/netbox/netbox/media/

## creates configuration file and populates with variables
echo creating netbox configuration file
cd /opt/netbox/netbox/netbox/

# copy configuration file from external location
# in this example I am copying this from an aws s3 bucket
aws s3 cp s3://S3BUCKET/Netbox/configuration.py ./configuration.py

sed -i "s/var_ipaddress/$var_ipaddress/g" configuration.py
sed -i "s/var_hostname/$var_hostname/g" configuration.py
sed -i "s/var_dbpassword/$var_dbpassword/g" configuration.py
sed -i "s/var_secretkey/$var_secretkey/g" configuration.py

## run upgrade script

## Does the following:
## Create a Python virtual environment
## Install all required Python packages
## Run database schema migrations
## Aggregate static resource files on disk

echo running netbox upgrade script
/opt/netbox/upgrade.sh
## restart of netbox &amp; netbox services is required here
sleep 30 
systemctl restart netbox netbox-rq

## gunicorn setup 
echo configuring gunicorn
cp /opt/netbox/contrib/gunicorn.py /opt/netbox/gunicorn.py

## systemd setup
echo configuring systemd
cp -v /opt/netbox/contrib/*.service /etc/systemd/system/
systemctl daemon-reload
systemctl start netbox netbox-rq
systemctl enable netbox netbox-rq

## nginx setup
echo configuring nginx and ssl
## generate self signed SSL certs

commonname=netbox
country=GB
state="City of Edinburgh"
locality=Edinburgh
organization=MyOrg
organizationalunit=IT
email=admin@admin.com

mkdir /etc/ssl/private
openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/ssl/private/netbox.key -out /etc/ssl/certs/netbox.crt -subj "/C=$country/ST=$state/L=$locality/O=$organization/OU=$organizationalunit/CN=$commonname/emailAddress=$email"

yum install -y nginx
cd /etc/nginx/conf.d/


cat &gt; netbox.conf &lt;&lt;EOF
server {
listen 443 ssl;

server_name netbox.MYDOMAIN.COM;

ssl_certificate /etc/ssl/certs/netbox.crt;
ssl_certificate_key /etc/ssl/private/netbox.key;

client_max_body_size 25m;

location /static/ {
    alias /opt/netbox/netbox/static/;
}

location / {
    proxy_pass http://127.0.0.1:8001;
    proxy_set_header X-Forwarded-Host \$http_host;
    proxy_set_header X-Real-IP \$remote_addr;
    proxy_set_header X-Forwarded-Proto \$scheme;
}
}

server {
# Redirect HTTP traffic to HTTPS
listen 80;
server_name netbox.MYDOMAIN.COM;
return 301 https://\$host\$request_uri;
}
EOF

sleep 30
systemctl restart nginx.service
systemctl enable nginx.service
sleep 10
systemctl is-active --quiet service nginx.service &amp;&amp; echo Service is running

echo database password = $var_dbpassword &gt;&gt; /root/netbox_creds.txt

echo install complete
echo credentials saved to /root/netbox_creds.txt
exit
</code></pre>
{% endhighlight %}
<!-- /wp:code -->
