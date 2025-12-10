Class-based views don’t magically solve N+1; they just give you structured places to fix it:

- `get_queryset()`
- `get_object()`
- `get_context_data()`
- and the `queryset` attribute

Below are several **common CBV scenarios** where N+1 happens, and the **exact fixes** for each.

I’ll reuse simple models:

```python
class Author(models.Model):
    name = models.CharField(max_length=255)

class Profile(models.Model):
    author = models.OneToOneField(Author, on_delete=models.CASCADE)
    bio = models.TextField()

class Publisher(models.Model):
    name = models.CharField(max_length=255)

class Book(models.Model):
    title = models.CharField(max_length=255)
    author = models.ForeignKey(Author, on_delete=models.CASCADE)
    publisher = models.ForeignKey(Publisher, on_delete=models.CASCADE)
    date_published = models.DateField()

class Tag(models.Model):
    name = models.CharField(max_length=50)
    books = models.ManyToManyField(Book, related_name='tags')
```

---

## Scenario 1 – `ListView` of books, template accesses `book.author`

**Symptom (N+1):**

```python
# views.py
class BookListView(ListView):
    model = Book
    template_name = 'books/home.html'
    context_object_name = 'books'
    ordering = ['-date_published']
```

```django
{# books/home.html #}
{% for book in books %}
  {{ book.title }} by {{ book.author.name }}
{% endfor %}
```

Generated queries:

1. `SELECT * FROM book ORDER BY date_published DESC;`
2. For each book in the loop, `SELECT * FROM author WHERE id = author_id;`

**Fix: override `get_queryset` and use `select_related`**

```python
class BookListView(ListView):
    model = Book
    template_name = 'books/home.html'
    context_object_name = 'books'
    ordering = ['-date_published']

    def get_queryset(self):
        return (
            super()
            .get_queryset()
            .select_related('author')          # FK => select_related
        )
```

Now Django does **one JOIN query** for books + authors.

---

## Scenario 2 – `ListView` of authors, template loops `author.book_set.all`

**Symptom (N+1 reverse FK):**

```python
class AuthorListView(ListView):
    model = Author
    template_name = 'authors/list.html'
    context_object_name = 'authors'
```

```django
{# authors/list.html #}
{% for author in authors %}
  <h2>{{ author.name }}</h2>
  <ul>
    {% for book in author.book_set.all %}
      <li>{{ book.title }}</li>
    {% endfor %}
  </ul>
{% endfor %}
```

Queries:

1. `SELECT * FROM author;`
2. For each author, `SELECT * FROM book WHERE author_id = <id>;`

**Fix: `prefetch_related` on reverse FK**

```python
class AuthorListView(ListView):
    model = Author
    template_name = 'authors/list.html'
    context_object_name = 'authors'

    def get_queryset(self):
        return (
            super()
            .get_queryset()
            .prefetch_related('book_set')      # or 'books' if you set related_name
        )
```

Now:

- 1 query for authors
- 1 query for all books for those authors  
→ **2 queries total**, regardless of number of authors.

---

## Scenario 3 – `ListView` including FK + ManyToMany + counts

You want:

- Book list page showing:
  - Book title
  - Author name
  - Publisher name
  - All tags
  - Number of tags

**Problematic version:**

```python
class BookListView(ListView):
    model = Book
    template_name = 'books/home.html'
    context_object_name = 'books'
```

```django
{% for book in books %}
  <h3>{{ book.title }}</h3>
  <p>By {{ book.author.name }} ({{ book.publisher.name }})</p>

  <p>Tags ({{ book.tags.count }}):</p>  {# count -> query per book #}
  <ul>
    {% for tag in book.tags.all %}
      <li>{{ tag.name }}</li>           {# tags.all -> another query per book #}
    {% endfor %}
  </ul>
{% endfor %}
```

N+1 on:

- `book.author`
- `book.publisher`
- `book.tags.count` / `.all()`

**Optimized CBV: `select_related` + `prefetch_related` + `annotate`**

```python
from django.db.models import Count

class BookListView(ListView):
    model = Book
    template_name = 'books/home.html'
    context_object_name = 'books'

    def get_queryset(self):
        return (
            Book.objects
            .select_related('author', 'publisher')  # FKs
            .prefetch_related('tags')              # M2M
            .annotate(tag_count=Count('tags'))     # avoid count() per book
            .order_by('-date_published')
        )
```

Template:

```django
{% for book in books %}
  <h3>{{ book.title }}</h3>
  <p>By {{ book.author.name }} ({{ book.publisher.name }})</p>

  <p>Tags ({{ book.tag_count }}):</p>
  <ul>
    {% for tag in book.tags.all %}
      <li>{{ tag.name }}</li>
    {% endfor %}
  </ul>
{% endfor %}
```

Now everything runs in a **small fixed number of queries** (books+authors+publishers in one, tags in one).

---

## Scenario 4 – `DetailView` of an author showing their books

Single object views can also suffer N+1 if you loop related items in the template.

**View:**

```python
class AuthorDetailView(DetailView):
    model = Author
    template_name = 'authors/detail.html'
    context_object_name = 'author'
```

**Template (N+1):**

```django
<h1>{{ author.name }}</h1>

<ul>
  {% for book in author.book_set.all %}
    <li>
      {{ book.title }} ({{ book.publisher.name }})
    </li>
  {% endfor %}
</ul>
```

Issues:

- Accessing `author.book_set.all` is just 1 query (fine).
- But inside that loop, accessing `book.publisher.name` can cause N+1 on publishers.

**Fix 1 – override `get_queryset()` and `select_related` via URL lookup**

If the author is looked up by `pk` or `slug`, we can customize `get_queryset`:

```python
class AuthorDetailView(DetailView):
    model = Author
    template_name = 'authors/detail.html'
    context_object_name = 'author'

    def get_queryset(self):
        # Preload books + their publishers for this author
        return (
            Author.objects
            .prefetch_related(
                'book_set__publisher'   # prefetch books, and select publishers
            )
        )
```

Now:

- 1 query for the author
- 1 query for all books (with publisher JOIN in that query)

**Fix 2 – use `get_object` + `prefetch_related`**

Another pattern:

```python
from django.shortcuts import get_object_or_404

class AuthorDetailView(DetailView):
    model = Author
    template_name = 'authors/detail.html'
    context_object_name = 'author'

    def get_object(self, queryset=None):
        qs = Author.objects.prefetch_related(
            'book_set__publisher'
        )
        return get_object_or_404(qs, pk=self.kwargs['pk'])
```

Both approaches avoid extra queries while iterating `author.book_set.all` and using `book.publisher`.

---

## Scenario 5 – `DetailView` of a book with deep relations

You want:

- Book detail page with:
  - Author
  - Author’s profile
  - Publisher
  - Tags

**Naive version:**

```python
class BookDetailView(DetailView):
    model = Book
    template_name = 'books/detail.html'
    context_object_name = 'book'
```

```django
<h1>{{ book.title }}</h1>
<p>By {{ book.author.name }}</p>
<p>Bio: {{ book.author.profile.bio }}</p>
<p>Publisher: {{ book.publisher.name }}</p>

<ul>
  {% for tag in book.tags.all %}
    <li>{{ tag.name }}</li>
  {% endfor %}
</ul>
```

On a **single object**, N+1 risk is mostly with:

- `book.tags.all` (M2M)
- if you later add loops over `book.reviews.all` or other related sets.

**Better: use `get_queryset()` with multi-level `select_related` + `prefetch_related`**

```python
class BookDetailView(DetailView):
    model = Book
    template_name = 'books/detail.html'
    context_object_name = 'book'

    def get_queryset(self):
        return (
            Book.objects
            .select_related(
                'author',
                'author__profile',
                'publisher',
            )
            .prefetch_related('tags')
        )
```

Now:

- `book.author`, `book.author.profile`, `book.publisher` are all in the main query.
- All tags are prefetched in a second query.

You’re safe to add more template logic accessing those relations.

---

## Scenario 6 – `ListView` with filters on related fields + prefetch

Example: You want a list of books filtered by author, and you also show all tags.

```python
class AuthorBookListView(ListView):
    model = Book
    template_name = 'books/by_author.html'
    context_object_name = 'books'

    def get_queryset(self):
        author_id = self.kwargs['author_id']
        return (
            Book.objects
            .filter(author_id=author_id)
            .order_by('-date_published')
        )
```

Template:

```django
{% for book in books %}
  {{ book.title }}
  <ul>
    {% for tag in book.tags.all %}
      <li>{{ tag.name }}</li>
    {% endfor %}
  </ul>
{% endfor %}
```

N+1 problem on `book.tags.all`.

**Fix: chain the filter with prefetch**

```python
def get_queryset(self):
    author_id = self.kwargs['author_id']
    return (
        Book.objects
        .filter(author_id=author_id)
        .select_related('author')   # if you render author info too
        .prefetch_related('tags')   # avoid N+1 for tags
        .order_by('-date_published')
    )
```

---

## Scenario 7 – `TemplateView` or custom CBV using multiple querysets

Homepage example with multiple lists:

```python
class HomePageView(TemplateView):
    template_name = 'home.html'

    def get_context_data(self, **kwargs):
        ctx = super().get_context_data(**kwargs)
        ctx['latest_books'] = Book.objects.order_by('-date_published')[:10]
        ctx['popular_authors'] = Author.objects.all()
        return ctx
```

Template:

```django
<h2>Latest Books</h2>
{% for book in latest_books %}
  {{ book.title }} by {{ book.author.name }}
{% endfor %}

<h2>Popular Authors</h2>
{% for author in popular_authors %}
  {{ author.name }} ({{ author.book_set.count }} books)
{% endfor %}
```

Problems:

- N+1 on `book.author` in first loop.
- N+1 on `author.book_set.count` in second loop.

**Fix: optimize each queryset in `get_context_data`**

```python
from django.db.models import Count

class HomePageView(TemplateView):
    template_name = 'home.html'

    def get_context_data(self, **kwargs):
        ctx = super().get_context_data(**kwargs)

        ctx['latest_books'] = (
            Book.objects
            .select_related('author')
            .order_by('-date_published')[:10]
        )

        ctx['popular_authors'] = (
            Author.objects
            .annotate(book_count=Count('book'))     # or Count('book_set')
            .prefetch_related('book_set')           # if you later show titles
            .order_by('-book_count')[:10]
        )

        return ctx
```

Template:

```django
<h2>Latest Books</h2>
{% for book in latest_books %}
  {{ book.title }} by {{ book.author.name }}
{% endfor %}

<h2>Popular Authors</h2>
{% for author in popular_authors %}
  {{ author.name }} ({{ author.book_count }} books)
{% endfor %}
```

No more N+1.

---

## Scenario 8 – `CreateView` / `UpdateView` + related lists in context

Form-based CBVs themselves don’t usually cause N+1, but **extra context** often does.

```python
class AuthorUpdateView(UpdateView):
    model = Author
    fields = ['name']
    template_name = 'authors/edit.html'
    context_object_name = 'author'

    def get_context_data(self, **kwargs):
        ctx = super().get_context_data(**kwargs)
        # All books for this author
        ctx['books'] = self.object.book_set.all()
        return ctx
```

Template:

```django
<form method="post">{% csrf_token %}
  {{ form.as_p }}
  <button type="submit">Save</button>
</form>

<h2>Books by this author</h2>
<ul>
  {% for book in books %}
    <li>{{ book.title }} ({{ book.publisher.name }})</li>
  {% endfor %}
</ul>
```

Potential N+1: `book.publisher.name` per book.

**Fix: prefetch in `get_object` or `get_queryset`**

Option A – override `get_object`:

```python
from django.shortcuts import get_object_or_404

class AuthorUpdateView(UpdateView):
    model = Author
    fields = ['name']
    template_name = 'authors/edit.html'
    context_object_name = 'author'

    def get_object(self, queryset=None):
        qs = Author.objects.prefetch_related('book_set__publisher')
        return get_object_or_404(qs, pk=self.kwargs['pk'])
```

Option B – override `get_queryset` (used by `get_object`):

```python
class AuthorUpdateView(UpdateView):
    model = Author
    fields = ['name']
    template_name = 'authors/edit.html'
    context_object_name = 'author'

    def get_queryset(self):
        return Author.objects.prefetch_related('book_set__publisher')
```

Now `.book_set.all()` and `book.publisher` are all preloaded.

---

## Scenario 9 – Abstracting optimization into a mixin

When you have many CBVs with the same optimization pattern, create a mixin.

Example: All book lists should eagerly load author and publisher.

```python
class BookQuerysetMixin:
    select_related_fields = ('author', 'publisher')
    prefetch_related_fields = ('tags',)

    def get_queryset(self):
        qs = super().get_queryset()
        if self.select_related_fields:
            qs = qs.select_related(*self.select_related_fields)
        if self.prefetch_related_fields:
            qs = qs.prefetch_related(*self.prefetch_related_fields)
        return qs
```

Use it:

```python
class BookListView(BookQuerysetMixin, ListView):
    model = Book
    template_name = 'books/home.html'
    context_object_name = 'books'
    ordering = ['-date_published']
```

```python
class BookByAuthorView(BookQuerysetMixin, ListView):
    model = Book
    template_name = 'books/by_author.html'
    context_object_name = 'books'

    def get_queryset(self):
        qs = super().get_queryset()
        return qs.filter(author_id=self.kwargs['author_id'])
```

This keeps N+1 fixes consistent and DRY.

---

## General rules for CBVs and N+1

1. **ListView**  
   Always ask: “What related fields do my templates access per item?”  
   Then put `select_related` / `prefetch_related` in `get_queryset`.

2. **DetailView / UpdateView / DeleteView**  
   If templates or `get_context_data` loop over related sets, optimize with:
   - `get_queryset` (preferred)  
   - or `get_object` using a custom queryset with prefetches.

3. **TemplateView / custom CBVs**  
   Any time you add lists in `get_context_data`, treat each list like a `ListView`:
   - Optimize its queryset before putting it in the context.

4. **Pattern recognition**  
   - Loop in template or view, and inside you access `.something_set.all` or `.fk_field` → likely N+1.
   - Fix at the CBV level by adjusting the queryset.

If you want, you can paste a specific CBV + template you’re working on, and I can rewrite it with the exact `select_related` / `prefetch_related` / `annotate` you need.