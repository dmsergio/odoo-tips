## Deploy (Nginx and SSL)

### How to install Nginx?
As *root*:
```shell
apt-get install nginx
```

### How to install certbot to generate free SSL certificates?
As *root*:
```shell
apt-get update
apt-get install sofware-properties-common
add-apt-repository universe
add-apt-repository ppa:certbot/certbot
apt-get update
apt-get install certbot python-certbot-nginx
```

### Generate certificate from [Let's Encrypt](https://letsencrypt.org/)
As *root*:
```shell
certbot certonly --standlone -n --agree-tos -m youremail@example.com -d your.domain.com
```

### Set up Nginx as a reverse proxy to handle encryption requests
- Create a configuration file in `/etc/nginx/sites-available/odoo.your.domain.com`.
- Fill the file using the template located in this folder (**nginx.template**).
- As *root* link the configuration file in `/etc/nginx/sites-enabled/`.
```shell
ln -s /etc/nginx/sites-available/odoo.your.domain.com /etc/nginx/sites-enabled/
```
- Enable proxy mode on Odoo config.
```editorconfig
proxy_mode = True
```
- As *root*, restart your odoo instance and nginx.
- As *root*, create a cron `/etc/cron.d/letsencrypt` file to renew the certificate.
```editorconfig
11 5 * * * certbot renew
```