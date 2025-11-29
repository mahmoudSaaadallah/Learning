### Introduction to CRUD Operations in Django ORM

The Django ORM provides a high-level, Pythonic interface to interact with your database. Instead of writing raw SQL for every data manipulation, you work with Python objects (your models). This abstraction not only speeds up development but also makes your code more readable and less prone to SQL injection vulnerabilities. The core operations—creating new data, modifying existing data, and removing data—are handled elegantly through model instances and QuerySets.

Let's continue using our familiar `Author` and `Book` models for demonstration:

```python
from django.db import models

class Author(models.Model):
    name = models.CharField(max_length=100)
    birth_date = models.DateField()

    def __str__(self):
        return self.name

class Book(models.Model):
    title = models.CharField(max_length=200)
    author = models.ForeignKey(Author, on_delete=models.CASCADE, related_name='books')
    publication_date = models.DateField()
    price = models.DecimalField(max_digits=6, decimal_places=2)
    pages = models.IntegerField()

    def __str__(self):
        return self.title
```

### 1. Creating Records

There are primarily two ways to create new records in Django.

#### Method 1: Using the `create()` method (Recommended for simplicity)

The `create()` method is a convenient one-liner that instantiates a model, saves it to the database, and returns the object.

```python
from datetime import date

# Create an Author
author1 = Author.objects.create(name='Jane Austen', birth_date=date(1775, 12, 16))
print(f"Created Author: {author1.name} (ID: {author1.id})")

# Create a Book, linking it to an existing Author
book1 = Book.objects.create(
    title='Pride and Prejudice',
    author=author1, # Pass the Author instance directly
    publication_date=date(1813, 1, 28),
    price=12.99,
    pages=400
)
print(f"Created Book: {book1.title} by {book1.author.name} (ID: {book1.id})")
```

**Key Point**: When creating a record with a `ForeignKey`, you pass the *instance* of the related object, not its primary key. Django handles the mapping to the foreign key ID in the database.

#### Method 2: Instantiating and then `save()`

This method gives you more control, allowing you to set attributes before saving. It's particularly useful when you need to perform some logic on the object before it hits the database.

```python
# Instantiate an Author object
author2 = Author(name='Charles Dickens', birth_date=date(1812, 2, 7))

# Perform some pre-save logic (e.g., validation, default value setting)
# ...

# Save the object to the database
author2.save()
print(f"Created Author: {author2.name} (ID: {author2.id})")

# Instantiate a Book object
book2 = Book(
    title='Great Expectations',
    author=author2,
    publication_date=date(1861, 8, 1),
    price=15.50,
    pages=500
)
book2.save()
print(f"Created Book: {book2.title} by {book2.author.name} (ID: {book2.id})")
```

**Important Note on `save()`**:
*   When `save()` is called on a new object, it performs an `INSERT` SQL statement.
*   When `save()` is called on an existing object (one that already has a primary key assigned, typically after being fetched from the database), it performs an `UPDATE` SQL statement.

### 2. Updating Records

Updating records can be done in two main ways: instance-level updates or QuerySet-level bulk updates.

#### Method 1: Updating a single instance

Fetch the object, modify its attributes, and then call `save()`.

```python
# Retrieve an existing book
book_to_update = Book.objects.get(title='Pride and Prejudice')
print(f"Original price of '{book_to_update.title}': ${book_to_update.price}")

# Modify an attribute
book_to_update.price = 14.99
book_to_update.pages = 420

# Save the changes to the database
book_to_update.save()
print(f"Updated price of '{book_to_update.title}': ${book_to_update.price}")
```

#### Method 2: Updating multiple instances with `update()`

The `update()` method allows you to update multiple objects in a QuerySet with a single database query, making it highly efficient. It does *not* call the `save()` method on each instance, meaning pre-save signals are not sent, and `auto_now` fields are not automatically updated.

```python
# Update all books by Jane Austen to have a higher price
jane_austen_books = Book.objects.filter(author__name='Jane Austen')
print(f"Books by Jane Austen before update: {[(b.title, b.price) for b in jane_austen_books]}")

# Perform the bulk update
updated_count = jane_austen_books.update(price=models.F('price') * 1.10) # Increase price by 10%
print(f"Updated {updated_count} books.")

# Verify the update (requires re-fetching or creating a new QuerySet due to QuerySet cache)
jane_austen_books_after = Book.objects.filter(author__name='Jane Austen')
print(f"Books by Jane Austen after update: {[(b.title, b.price) for b in jane_austen_books_after]}")
```

**Using `F()` expressions for atomic updates**:
Notice the use of `models.F('price')` in the example above. `F()` expressions allow you to refer to model field values directly in the database, rather than as Python values. This is crucial for atomic updates, preventing race conditions where two processes might try to update the same field simultaneously.

```python
# Increment the page count for a specific book atomically
book_to_increment = Book.objects.get(title='Great Expectations')
print(f"Original pages: {book_to_increment.pages}")

Book.objects.filter(id=book_to_increment.id).update(pages=models.F('pages') + 10)

book_to_increment.refresh_from_db() # Get the latest data from the database
print(f"New pages: {book_to_increment.pages}")
```

### 3. Deleting Records

Deletion, like creation and update, can be performed on single instances or multiple instances via a QuerySet.

#### Method 1: Deleting a single instance

Fetch the object and call its `delete()` method.

```python
# Create a temporary author and book for deletion demo
temp_author = Author.objects.create(name='Temporary Author', birth_date=date(2000, 1, 1))
temp_book = Book.objects.create(
    title='Temporary Book',
    author=temp_author,
    publication_date=date(2020, 1, 1),
    price=9.99,
    pages=100
)
print(f"Created temporary book: {temp_book.title} (ID: {temp_book.id})")

# Delete the book
temp_book.delete()
print(f"Deleted temporary book. Does it exist? {Book.objects.filter(id=temp_book.id).exists()}")

# Note: The author still exists unless on_delete=CASCADE was set on the ForeignKey
# and the author had no other books.
```

The `delete()` method returns a tuple: `(number_of_objects_deleted, a_dictionary_of_deletions_per_model_type)`.

#### Method 2: Deleting multiple instances with `delete()` on a QuerySet

This is the most efficient way to delete multiple objects, as it performs a single SQL `DELETE` statement.

```python
# Let's create a few more books by our temporary author
Book.objects.create(title='Another Temp Book', author=temp_author, publication_date=date(2021,1,1), price=10.00, pages=120)
Book.objects.create(title='Third Temp Book', author=temp_author, publication_date=date(2022,1,1), price=11.00, pages=130)

# Now, delete all books by the temporary author
books_to_delete = Book.objects.filter(author=temp_author)
print(f"Books by Temporary Author before deletion: {books_to_delete.count()}")

deleted_info = books_to_delete.delete()
print(f"Deleted info: {deleted_info}")
print(f"Books by Temporary Author after deletion: {Book.objects.filter(author=temp_author).count()}")

# If the ForeignKey to Author had on_delete=CASCADE, and this was the last book,
# the author might also be deleted. Let's check.
# In our model, Book.author has on_delete=models.CASCADE.
# If temp_author had no other books, deleting its books would also delete temp_author.
# Let's delete the author directly now that its books are gone.
temp_author.delete()
print(f"Temporary Author exists? {Author.objects.filter(id=temp_author.id).exists()}")
```

**Understanding `on_delete` behavior**:
This is critical for `ForeignKey` and `OneToOneField` relationships. The `on_delete` argument specifies what happens to the related object when the referenced object is deleted.

*   `models.CASCADE` (default): Deletes the object containing the `ForeignKey` when the referenced object is deleted. (e.g., delete an Author, all their Books are deleted).
*   `models.PROTECT`: Prevents deletion of the referenced object if it has related objects. (e.g., cannot delete an Author if they still have Books).
*   `models.SET_NULL`: Sets the `ForeignKey` to `NULL` when the referenced object is deleted. Requires `null=True` on the `ForeignKey`. (e.g., delete an Author, their Books' `author` field becomes `NULL`).
*   `models.SET_DEFAULT`: Sets the `ForeignKey` to its default value when the referenced object is deleted. Requires a `default` value on the `ForeignKey`.
*   `models.DO_NOTHING`: Does nothing. This can lead to data integrity issues if not handled carefully.
*   `models.SET()`: Sets the `ForeignKey` to the value passed to `SET()`, or to the result of a callable.

### 4. Bulk Operations (Advanced Efficiency)

For very large datasets, Django offers `bulk_create` and `bulk_update` for even greater efficiency by performing operations in a single query.

*   **`bulk_create(objs, batch_size=None)`**: Creates multiple objects in a single query. It does not call `save()` on each object, so `pre_save`/`post_save` signals are not sent, and `auto_now_add` fields are not automatically set.
```python
new_books = [
	Book(title='Book A', author=author1, publication_date=date(2023,1,1), price=20.00, pages=200),
	Book(title='Book B', author=author1, publication_date=date(2023,2,1), price=25.00, pages=250),
]
Book.objects.bulk_create(new_books)
print(f"Bulk created {len(new_books)} books.")
```

*   **`bulk_update(objs, fields, batch_size=None)`**: Updates multiple existing objects in a single query. You must provide a list of fields to update. Like `bulk_create`, it bypasses `save()` and its associated signals.
```python
# Assume we have some books to update
books_to_modify = list(Book.objects.filter(author=author1, price__lt=30)) # Fetch existing books
for book in books_to_modify:
	book.price += 5.00 # Modify in Python

Book.objects.bulk_update(books_to_modify, ['price'])
print(f"Bulk updated prices for {len(books_to_modify)} books.")
```

### Conclusion

Mastering the CRUD operations in Django ORM is fundamental. Whether you're creating a new user, updating an order status, or deleting an old product, the ORM provides powerful and flexible tools. Understanding the nuances of `save()`, `create()`, `update()`, `delete()`, and especially the implications of `on_delete` and the efficiency gains of `F()` expressions and bulk operations, will enable you to write highly effective, performant, and robust Django applications. Always consider the database implications of your ORM calls, and you'll be well on your way to becoming an ORM virtuoso.