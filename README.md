# Odoo tips

### Database tips

**Creating a backup (with attachments)**
```{shell}
pg_dump -Fc -f {database}.dump {database}
tar cjf {database}.tgz {database}.dump ~/.local/share/Odoo/filestore/{database}
```

**Restoring a backup in a new database**
```{sql}
CREATE DATABASE {database} WITH OWNER odoo TEMPLATE template1;
```
```{shell}
tar xf {database}.tgz
pg_restore -C -d {database} {database}.dump
cp -r ./home/odoo/.local/share/odoo/filestore{database} ~/.local/share/Odoo/filestore/
```

### Odoo ORM tips
