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

### Examples
```python
from odoo import exceptions
from odoo.tests import common


class TestFoo(common.TransactionCase):
    def test_assert_equal(self):
        result = self.function_to_test()
        expected = ...  # expected result
        self.assertEqual(result, expected)

    def test_assert_not_equal(self):
        result = self.function_to_test()
        not_expected = ...  # not expected result
        self.assertNotEqual(result, not_expected)

    def test_assert_true(self):
        result = self.function_to_test()
        self.assertTrue(result)

    def test_assert_false(self):
        result = self.function_to_test()
        self.assertFalse(result)

    def test_assert_in(self):
        result = self.function_to_test()
        expected_values = ...  # expected values
        self.assertIn(result, expected_values)

    def test_assert_not_in(self):
        result = self.function_to_test()
        not_expected_values = ...
        self.assertIn(result, not_expected_values)

    def test_assert_raises(self):
        with self.assertRaises(exceptions.UserError) as e:
            self.function_to_test()  # this function must be to raise an exception
            # in addition, is possible assert the error message
            expected_error = ...
            self.assertEqual(e.name, expected_error)

    def test_assert_is_none(self):
        result = self.function_to_test()
        self.assertIsNone(result)

    def test_assert_is_not_none(self):
        result = self.function_to_test()
        self.assertIsNotNone(result)

    def test_assert_is_instance(self):
        result = self.function_to_test()
        expected_instance = ...
        self.assertIsInstance(result, expected_instance)
```
