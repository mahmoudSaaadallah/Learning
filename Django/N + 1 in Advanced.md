The *N + 1 problem* in Django ORM is a performance anti‑pattern where instead of doing **1** query (or a small constant number), your code does **N + 1** queries:  

- 1 query to get a list of objects,  
- then N extra queries to fetch related data for each object in that list.

In small datasets you don’t notice; at scale it becomes catastrophic.

Below I’ll:

1. Formalize the idea of N+1 in Django.
2. Show **many concrete examples** (forward FK, reverse FK, ManyToMany, templates, admin, DRF, etc.).
3. Show how to fix each with `select_related`, `prefetch_related`, and other patterns.

---

## 1. Minimal model setup we’ll reuse

We’ll use these models; assume `app_name = "library"`:

```python
# library/models.py

from django.db import models

class Publisher(models.Model):
    name = models.CharField(max_length=255)

class Author(models.Model):
    name = models.CharField(max_length=255)

class Profile(models.Model):
    author = models.OneToOneField(Author, on_delete=models.CASCADE)
    bio = models.TextField()

class Book(models.Model):
    title = models.CharField(max_length=255)
    author = models.ForeignKey(Author, on_delete=models.CASCADE)
    publisher = models.ForeignKey(Publisher, on_delete=models.CASCADE)
    published_at = models.DateField()
    pages = models.IntegerField()

class Tag(models.Model):
    name = models.CharField(max_length=50)
    books = models.ManyToManyField(Book, related_name='tags')

class Review(models.Model):
    book = models.ForeignKey(Book, on_delete=models.CASCADE, related_name='reviews')
    rating = models.IntegerField()
    comment = models.TextField()
```

We’ll build examples on top of these.

---

## 2. Basic N+1 with a forward ForeignKey

### Example 1 – Fetching Books then accessing `book.author`

```python
# views.py
from django.shortcuts import render
from .models import Book

def book_list(request):
    books = Book.objects.all()  # Query 1  => Query set
    for book in books:
        print(book.title, book.author.name)  # Potential N extra queries
    return render(request, 'books.html', {'books': books})
```

**SQL behavior** (conceptually):

1. `SELECT * FROM library_book;`  → 1 query to fetch all books.
2. For each `book`:
   - Accessing `book.author` triggers:  
     `SELECT * FROM library_author WHERE id = <book.author_id>;`
   - That happens N times (once per book).

So you end up with:

- **1** query for books
- **N** queries for authors  
= **N + 1** queries total.

### Fix with `select_related`

Use `select_related` for **ForeignKey** and **OneToOneField** (single-valued relationships):

```python
def book_list(request):
    books = Book.objects.select_related('author').all()
    for book in books:
        print(book.title, book.author.name)  # No extra queries
    return render(request, 'books.html', {'books': books})
```

Now:

- Django does one JOIN query:

  ```sql
  SELECT
      "library_book"."id",
      "library_book"."title",
      "library_book"."author_id",
      "library_author"."id",
      "library_author"."name"
  FROM "library_book"
  INNER JOIN "library_author"
      ON ("library_book"."author_id" = "library_author"."id");
  ```

- **1 query total**, no N+1.

---

## 3. N+1 with deeper relations (chain of FKs / OneToOne)

### Example 2 – `book.author.profile.bio`

```python
def book_list(request):
    books = Book.objects.all()
    for book in books:
        print(book.title, book.author.profile.bio)
```

Queries:

1. `SELECT * FROM library_book;` (all books)
2. For each book:
   - Access `book.author` → query per book (unless cached).
   - Access `book.author.profile` → query per unique author (unless cached).

In worst case, for N books and N authors and N profiles, you can see ~2N additional queries.

### Fix with multi-level `select_related`

```python
def book_list(request):
    books = (
        Book.objects
        .select_related('author__profile')
        .all()
    )
    for book in books:
        print(book.title, book.author.profile.bio)
```

One query with multiple JOINs:

```sql
SELECT ...
FROM "library_book"
INNER JOIN "library_author" ON ...
LEFT OUTER JOIN "library_profile" ON ...
```

---

## 4. N+1 with reverse ForeignKey (One-to-Many)

### Example 3 – Author with books (`author.book_set.all()`)

```python
def author_list(request):
    from .models import Author
    authors = Author.objects.all()  # Query 1

    for author in authors:
        # Reverse FK lookup: book_set
        books = author.book_set.all()  # Potential query per author
        print(author.name, [b.title for b in books])
```

Queries:

1. `SELECT * FROM library_author;`
2. For each author:
   - `SELECT * FROM library_book WHERE author_id = <author.id>;`  

Total: **1 + N** queries.

### Fix with `prefetch_related`

For reverse FKs (and M2M), use `prefetch_related`:

```python
from django.db.models import Prefetch

def author_list(request):
    authors = (
        Author.objects
        .prefetch_related('book_set')  # or 'books' if you used related_name
    )

    for author in authors:
        books = author.book_set.all()  # No new queries; uses cache
        print(author.name, [b.title for b in books])
```

Django does:

1. `SELECT * FROM library_author;`
2. `SELECT * FROM library_book WHERE author_id IN (list of author ids);`

Still 2 queries total, regardless of N authors.

---

## 5. N+1 with ManyToMany relations

### Example 4 – Books with tags (`book.tags.all()`)

```python
def book_tags(request):
    books = Book.objects.all()  # Query 1

    for book in books:
        tags = book.tags.all()  # Query per book
        print(book.title, [t.name for t in tags])
```

Queries:

1. `SELECT * FROM library_book;`
2. For each book:

   ```sql
   SELECT "library_tag".*
   FROM "library_tag"
   INNER JOIN "library_tag_books" ON ("library_tag"."id" = "library_tag_books"."tag_id")
   WHERE "library_tag_books"."book_id" = <book.id>;
   ```

Total: **1 + N** queries.

### Fix with `prefetch_related` on M2M

```python
def book_tags(request):
    books = Book.objects.prefetch_related('tags')  # Query 1 + 1

    for book in books:
        tags = book.tags.all()  # Uses prefetch cache
        print(book.title, [t.name for t in tags])
```

Django does:

1. `SELECT * FROM library_book;`
2. `SELECT * FROM library_tag INNER JOIN library_tag_books ... WHERE book_id IN (...);`

Total: **2** queries.

### Example 5 – ManyToMany from Tag side (`tag.books.all()`)

```python
def tag_books(request):
    tags = Tag.objects.all()  # Query 1
    for tag in tags:
        books = tag.books.all()  # Query per tag
        print(tag.name, [b.title for b in books])
```

Same N+1 structure; fix:

```python
def tag_books(request):
    tags = Tag.objects.prefetch_related('books')
    for tag in tags:
        books = tag.books.all()  # No extra queries
        print(tag.name, [b.title for b in books])
```

---

## 6. N+1 in templates (very common)

N+1 often hides in templates, because you write “simple” loops.

### Example 6 – Template N+1 on `book.author`

**View:**

```python
def book_list(request):
    books = Book.objects.all()  # No optimization
    return render(request, 'book_list.html', {'books': books})
```

**Template (`book_list.html`):**

```django
{% for book in books %}
  <p>{{ book.title }} by {{ book.author.name }}</p>
{% endfor %}
```

This is the same N+1 pattern as Example 1, but moved into the template.

**Fix:**

```python
def book_list(request):
    books = Book.objects.select_related('author')
    return render(request, 'book_list.html', {'books': books})
```

Same idea for reverse/many-to-many:

### Example 7 – Template N+1 on `author.book_set.all`

**View:**

```python
def author_list(request):
    authors = Author.objects.all()
    return render(request, 'author_list.html', {'authors': authors})
```

**Template:**

```django
{% for author in authors %}
  <h3>{{ author.name }}</h3>
  <ul>
    {% for book in author.book_set.all %}
      <li>{{ book.title }}</li>
    {% endfor %}
  </ul>
{% endfor %}
```

That `author.book_set.all` makes a query per author.

**Fix:**

```python
def author_list(request):
    authors = Author.objects.prefetch_related('book_set')
    return render(request, 'author_list.html', {'authors': authors})
```

---

## 7. N+1 in Django Admin

The admin site can hit N+1 when listing objects with related fields in `list_display` or showing inlines.

### Example 8 – Admin `list_display` using FK field

```python
# admin.py
from django.contrib import admin
from .models import Book

@admin.register(Book)
class BookAdmin(admin.ModelAdmin):
    list_display = ('title', 'author', 'publisher')
```

By default, listing many books may cause many extra queries to display `author` and `publisher`.

**Fix with `get_queryset` override and `select_related`:**

```python
@admin.register(Book)
class BookAdmin(admin.ModelAdmin):
    list_display = ('title', 'author', 'publisher')

    def get_queryset(self, request):
        qs = super().get_queryset(request)
        return qs.select_related('author', 'publisher')
```

For reverse relations or M2M in list_display, consider `prefetch_related`.

---

## 8. N+1 in Django REST Framework (DRF) serializers

Nested serializers are very prone to N+1.

### Example 9 – DRF N+1 with nested author serializer

```python
# serializers.py
from rest_framework import serializers
from .models import Book, Author

class AuthorSerializer(serializers.ModelSerializer):
    class Meta:
        model = Author
        fields = ('id', 'name')

class BookSerializer(serializers.ModelSerializer):
    author = AuthorSerializer()

    class Meta:
        model = Book
        fields = ('id', 'title', 'author')
```

**View:**

```python
from rest_framework.generics import ListAPIView

class BookListAPIView(ListAPIView):
    queryset = Book.objects.all()  # N+1 risk
    serializer_class = BookSerializer
```

DRF will serialize each book’s `.author`, generating N queries (one per book).

**Fix:**

```python
class BookListAPIView(ListAPIView):
    queryset = Book.objects.select_related('author')
    serializer_class = BookSerializer
```

### Example 10 – DRF N+1 with reverse relation (Author → Books)

```python
class BookShortSerializer(serializers.ModelSerializer):
    class Meta:
        model = Book
        fields = ('id', 'title')

class AuthorWithBooksSerializer(serializers.ModelSerializer):
    books = BookShortSerializer(source='book_set', many=True)

    class Meta:
        model = Author
        fields = ('id', 'name', 'books')
```

If you use:

```python
class AuthorListAPIView(ListAPIView):
    queryset = Author.objects.all()  # N+1
    serializer_class = AuthorWithBooksSerializer
```

You hit N+1 on `author.book_set.all()`.

**Fix:**

```python
class AuthorListAPIView(ListAPIView):
    queryset = Author.objects.prefetch_related('book_set')
    serializer_class = AuthorWithBooksSerializer
```

You can also use `Prefetch` to limit or order related objects:

```python
from django.db.models import Prefetch

class AuthorListAPIView(ListAPIView):
    queryset = Author.objects.prefetch_related(
        Prefetch('book_set', queryset=Book.objects.order_by('-published_at'))
    )
    serializer_class = AuthorWithBooksSerializer
```

---

## 9. N+1 in aggregations / per-object computations

### Example 11 – Counting related objects in a loop

```python
def book_review_counts(request):
    books = Book.objects.all()
    data = []
    for book in books:
        count = book.reviews.count()  # Query per book
        data.append((book.title, count))
```

For N books, you get N extra `COUNT()` queries:

```sql
SELECT COUNT(*) FROM "library_review" WHERE "book_id" = <book.id>;
```

**Fix with `annotate` + `Count`**

```python
from django.db.models import Count

def book_review_counts(request):
    books = Book.objects.annotate(review_count=Count('reviews'))
    data = [(book.title, book.review_count) for book in books]  # 1 query total
```

Annotation moves the computation to a single query:

```sql
SELECT library_book.*, COUNT(library_review.id) AS review_count
FROM library_book
LEFT OUTER JOIN library_review ON ...
GROUP BY library_book.id;
```

### Example 12 – N+1 with `.exists()` per object

```python
def book_has_reviews(request):
    books = Book.objects.all()
    data = []
    for book in books:
        has_reviews = book.reviews.exists()  # Query per book
        data.append((book.title, has_reviews))
```

Again, N extra queries.

Depending on what you need, you can:

- Use `annotate(Count('reviews'))` and check `review_count > 0`, or
- Use `Prefetch` and manually check length:

```python
from django.db.models import Prefetch

def book_has_reviews(request):
    books = Book.objects.prefetch_related(
        Prefetch('reviews', queryset=Review.objects.only('id'))
    )
    for book in books:
        has_reviews = len(book.reviews.all()) > 0
```

Still 2 queries total.

---

## 10. Combining `select_related` and `prefetch_related`

### Example 13 – Books with author, publisher, and tags

```python
def full_book_list(request):
    books = (
        Book.objects
        .select_related('author', 'publisher')   # FK & O2O
        .prefetch_related('tags', 'reviews')     # M2M & reverse FK
    )
    for book in books:
        print(
            book.title,
            book.author.name,
            book.publisher.name,
            [t.name for t in book.tags.all()],
            [r.rating for r in book.reviews.all()],
        )
```

Queries:

1. Single JOIN query for `Book + Author + Publisher`.
2. Prefetch query for `Tag` via M2M.
3. Prefetch query for `Review` via reverse FK.

Still **3 queries total**, no matter how many books.

---

## 11. More subtle examples

### Example 14 – Using `values()` can hide N+1

```python
def book_data(request):
    books = Book.objects.values('id', 'title')  # Only book fields
    for b in books:
        # Now you want the author name:
        author = Author.objects.get(id=b['author_id'])  # You don't have author_id actually
```

Often, people:

1. Use `values()` to “optimize”.
2. Then need related data and do separate lookups in a loop → N+1.

Better:

```python
books = (
    Book.objects
    .select_related('author')
    .values('id', 'title', 'author__name')
)
```

Single query with JOIN.

### Example 15 – Signals creating N+1 in background

Suppose you have a `post_save` signal for `Review` that recalculates average rating per book:

```python
from django.db.models.signals import post_save
from django.dispatch import receiver
from django.db.models import Avg
from .models import Review

@receiver(post_save, sender=Review)
def update_book_rating(sender, instance, **kwargs):
    book = instance.book
    avg = book.reviews.aggregate(Avg('rating'))['rating__avg']  # Query each time
    book.average_rating = avg
    book.save()
```

If you bulk-create reviews or import many, this might cause N+1 or worse. You might prefer:

- Bulk operations (calculate once for a group), or
- Use `F()` expressions / denormalized counters, etc.

---

## 12. How to *see* N+1 in practice

### Django Debug Toolbar

Install `django-debug-toolbar` and look at the **SQL panel**. On a page that lists lots of objects:

- If you see 200, 300, 1000 queries for one page load, you likely have N+1 problems.
- Click each query: you’ll see repeating patterns like:

  ```sql
  SELECT * FROM library_author WHERE id = 1;
  SELECT * FROM library_author WHERE id = 2;
  SELECT * FROM library_author WHERE id = 3;
  ...
  ```

### `connection.queries`

For simple debugging in a view:

```python
from django.db import connection, reset_queries
from django.conf import settings

def debug_view(request):
    if settings.DEBUG:
        reset_queries()

    books = Book.objects.all()
    for book in books:
        print(book.author.name)

    if settings.DEBUG:
        print(len(connection.queries))  # Number of SQL queries
        for q in connection.queries:
            print(q['sql'])
```

### `.explain()` on QuerySets

`Book.objects.select_related('author').explain()` can help you see the query plan (not directly about N+1, but still useful for serious optimization).

---

## 13. Key mental model & rules of thumb

1. **N+1 Pattern**: Any time you:
   - Fetch a list,
   - And inside a loop or template, access related objects that weren’t preloaded,
   - You’re likely doing N+1.

2. **Use `select_related`**:
   - For **ForeignKey** and **OneToOneField**.
   - Works only for single-valued relationships.
   - Follows relationships via SQL JOIN.

3. **Use `prefetch_related`**:
   - For **ManyToMany** and **reverse ForeignKey**.
   - Executes separate queries and combines results in Python.
   - Works for anything returning a QuerySet (`.all()`, reverse relations, etc.).

4. **Inspect queries**:
   - Debug Toolbar is your friend.
   - If the number of queries grows linearly with the number of objects displayed → N+1.

5. **Be extra careful in templates, serializers, and admin**:
   - N+1 tends to hide there because your Python code looks fine and queries are implicit.

---

If you’d like, I can:

- Take a specific view/template/serializer of yours and rewrite it to remove N+1, or  
- Show more advanced patterns, like conditional `Prefetch`, custom related names, or optimizing nested serializers with many layers.