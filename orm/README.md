## ORM

### Model attributes

- **_name**: Define the internal global identifier. Odoo creates the database table with this name.
- **_description**: User friendly title to the model.
- **_order**: Sort records by fields.
- **_rec_name**: Record representation.

```python
from odoo import fields, models

class MyFoo(models.Model):
    _name = "my.foo"
    _description = "My Foo"
    _order = "release_date desc, title"
    _rec_name = "title"
    _log_access = True  # is True by default. If False, create and write audit logs will not be generated

    name = fields.Char("Name", required=True)  # by default will be used to represent a record of this model
    title = fields.Char("Title", required=True)
    release_date = fields.Date("Release date")
```
