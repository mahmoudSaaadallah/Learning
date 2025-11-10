That's a great question about Django model fields, and it touches upon how we manage timestamps in our database records. Even though the [[Class Based View]] note uses these in its `Product` model, let's clarify their exact behavior.

In Django's `DateTimeField`, `auto_now_add` and `auto_now` are two very useful parameters, but they serve distinct purposes:

### `auto_now_add=True`

*   **Purpose:** This automatically sets the field's value to the current datetime **only when the object is first created**.
*   **Behavior:** Once set, this value **cannot be changed** through the Django ORM. It's essentially a "creation timestamp."
*   **Use Case:** Ideal for fields like `created_at`, `date_joined`, or `published_on` where you want to record *when* an instance was initially added to the database and ensure that timestamp remains immutable.

**Example:**

```python
from django.db import models

class Product(models.Model):
    name = models.CharField(max_length=255)
    # This field will be set once when a new Product is saved, and never change.
    created_at = models.DateTimeField(auto_now_add=True) 

    def __str__(self):
        return self.name

# When you create a product:
product = Product.objects.create(name="New Gadget")
print(product.created_at) # Will show the current datetime

# If you try to update it later:
product.name = "Updated Gadget"
product.save()
print(product.created_at) # Will still show the *original* creation datetime
```

### `auto_now=True`

*   **Purpose:** This automatically updates the field's value to the current datetime **every time the object is saved**.
*   **Behavior:** This value will change on both the initial creation *and* any subsequent updates to the object.
*   **Use Case:** Perfect for fields like `updated_at`, `last_modified`, or `last_login` where you want to track *when* an instance was last modified.

**Example:**

```python
from django.db import models
import time

class Product(models.Model):
    name = models.CharField(max_length=255)
    # This field will be updated every time the Product instance is saved.
    updated_at = models.DateTimeField(auto_now=True) 

    def __str__(self):
        return self.name

# When you create a product:
product = Product.objects.create(name="New Gadget")
print(product.updated_at) # Will show the current datetime

# Wait a moment, then update it:
time.sleep(1) # Simulate some time passing
product.name = "Updated Gadget"
product.save()
print(product.updated_at) # Will show the *new* current datetime (when it was saved again)
```

### Key Differences Summarized:

| Feature          | `auto_now_add=True`                               | `auto_now=True`                                   |
| :--------------- | :------------------------------------------------ | :------------------------------------------------ |
| **When Set**     | Only on **creation** of the object.               | On **creation** AND **every subsequent save**.    |
| **Mutability**   | Value is **immutable** after initial creation.    | Value **changes** with every save operation.      |
| **Common Use**   | `created_at`, `date_joined`, `first_seen`.        | `updated_at`, `last_modified`, `last_login`.      |
| **Manual Edit**  | Cannot be manually set or changed via ORM.        | Cannot be manually set or changed via ORM.        |

In the `Product` model example from the [[Class Based View]] note, you can see both in action:

```python
class Product(models.Model):
    # ... other fields ...
    created_at = models.DateTimeField(auto_now_add=True) # Set once on creation
    updated_at = models.DateTimeField(auto_now=True)     # Updates on every save
    # ...
```

Understanding this distinction is crucial for maintaining accurate historical data and for implementing common API patterns like showing when a resource was created versus when it was last modified. Excellent question, keep them coming!