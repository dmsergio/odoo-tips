# Odoo tips

## Database tips

### Creating a backup (with attachments)
```shell
pg_dump -Fc -f {database}.dump {database}
tar cjf {database}.tgz {database}.dump ~/.local/share/Odoo/filestore/{database}
```

### Restoring a backup in a new database
```sql
CREATE DATABASE {database} WITH OWNER='odoo' TEMPLATE='template1' ENCODING='UTF8';
```
```shell
tar xf {database}.tgz
pg_restore -C -d {database} {database}.dump
cp -r ./home/odoo/.local/share/odoo/filestore{database} ~/.local/share/Odoo/filestore/
```

---

## Odoo config file

### Workers
- 1 worker ~= 6 concurrent users
- Rule of thumb: #workers = (#CPU * 2) + 1
- Example of #CPU with an Odoo instance with 60 concurrent users:
  - 60 concurrent users / 6 = 10 workers
  - CPU ~= 5
  - (5 * 2) + 1 = 11

**limit_request (8192 by default)**
- A worker will be terminated after having processed this many requests.

**limit_memory_hard (Odoo recommends 4GB)** (Set values in bytes. 1GB = 1024 MB = 1.048.576 KB = 1.073.741.824 B)
- Maximum amount of RAM a worker will be able to allocate. If a worker exceeds, Odoo will kill it immediately.

**limit_memory_soft** (Set values in bytes. 1GB = 1024 MB = 1.048.576 KB = 1.073.741.824 B)
- If a worker ends up consuming more than this limit, it will be terminated after it finishes processing the current request.

**limit_time_cpu** (Set values in seconds)
- The maximum amount of CPY time allowed to process a request.

**limit_time_real** (Set values in seconds)
- The maximum amount of wall-clock time allowed to process a request. Differs from limit_time_cpu in that this is a "wall time" limit including SQL queries.

### Logs

**logrotate = True**
- This will cause Odoo to configure the logging module to archive the server logs on a daily bases, and to keep the old logs for 30 days.

---

## Nginx and SSL

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
- Use the next template to fill the new file:
```editorconfig
upstream odoo {
    server 127.0.0.1:8069;
}
upstream odoochat {
    server 127.0.0.1:8072;
}
server {
    listen 80;
    server_name your.domain.com;
    rewrite ^(.*) https://$host$1 permanent;
}
server {
    listen 443;
    server_name your.domain.com;
    proxy_read_timeout 720s;
    proxy_connect_timeout 720s;
    proxy_send_timeout 720s;
    
    # SSL Configuration
    ssl on;
    ssl_certificate /etc/letsencrypt/live/your.domain.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/your.domain.com/privkey.pem;
    ssl_session_timeout 30m;
    ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
    ssl_ciphers "ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384";
    ssl_prefer_server_ciphers on;
    
    # Add log files
    access_log /var/log/nginx/your.domain.access.log;    
    error_log /var/log/nginx/your.domain.error.log;
    
    # Enable gzip
    gzip on;
    gzip_types text/css text/scss text/plain text/xml application/xml application/json application/javascript;
    
    # Add Headers
    proxy_set_header X-Forwarded-Host $host;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;
    proxy_set_header X-Real-IP $remote_addr;
    
    # Manage longpolling on 8072 port
    location /longpolling {
        proxy_pass http://odoochat;
    }
    
    # Redirect requests to odoo server on 8069
    location / {
        proxy_redirect off;
        proxy_pass http://odoo;
    }
    
    # Enable static cache
    location ~* /web/static/ {
        proxy_cache_valid 200 60m;
        proxy_buffering on;
        expires 864000;
        proxy_pass http://odoo;
    }
}
```
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

---

## Tests

### Types

All the test classes are located on `odoo.tests.common.py` file.

- **TransactionCase**: TestCase in which each test method is run in its own transaction,
    and with its own cursor. The transaction is rolled back and the cursor
    is closed after each test.
- **SingleTransactionCase**: TestCase in which all test methods are run in the same transaction,
    the transaction is started with the first test method and rolled back at
    the end of the last.
- **SavepointCase**: Similar to SingleTransactionCase in that all test methods
    are run in a single transaction *but* each test case is run inside a
    rollbacked savepoint (sub-transaction).
- **HttpCase**: Transactional HTTP TestCase with url_open and Chrome headless helpers.

### Set up

- **setUp function**: Allow set params or configs to can be available in all test functions. It will be invoked
    for each test function inside the class. E.g.:
    ```python
    from odoo.tests import common
    
  
    class TestFoo(common.TransactionCase):
        def setUp(self):
            super(TestFoo, self).setUp()
            self.Model = self.env["res.user"]
  
        def test_foo1(self):
            user = self.Model.search([...])
    ```

### Tags

There is a decorator available in `odoo.tests.common` called **tagged**, which allow set tags to our both test classes
and test functions to modify behavior when the tests are executed.

Tags available:

- **standard**: Execute all tests defined inside the addon to install/update.
- **at_install**: The test is executed once the addon has been installed/updated.
- **post_install**: The test is executed once all the addon dependencies has been installed/updated.
- **Custom tags**: Is possible add custom tags, which will be possible to invoke it from command line tests run.

By default, the tags **standard** and **at_install** are used by Odoo tests, so if you don't add any tag, this two
will be added automatically.

Is possible remove default tags, like **at_install**, with the "-" prefix.

Examples:

```python
from odoo.tests import common, tagged

@tagged("-at_install", "post_install")
class TestFoo1(common.TransactionCase):
    def test_foo1(self):
        ...

@tagged("-standard", "my_custom_tag")
class TestFoo2(common.TransactionCase):
    def test_foo2(self):
        ...
```

### How to run tests?
```shell
# run all tests defined in the addon
odoo -d my_db -i my_addon --test-enable --log-level=test --stop-after-init

# run only the tests defined in the class TestClass
odoo -d my_db -i my_addon --test-tags:TestClass --stop-after-init

# run only the test test_function defined in the class TestClass
odoo -d my_db -i my_addon --test-tags:TestClass.test_function --stop-after-init

# run the two functions tests defined in the class TestClass
odoo -d my_db -i my_addon --test-tags:TestClass.test_function,:TestClass.test_function2 --stop-after-init

# only run tests decorated with `my_custom_tag` tag
odoo -d my_db -i my_addon --test-tags:my_custom_tag --stop-after-init
```
