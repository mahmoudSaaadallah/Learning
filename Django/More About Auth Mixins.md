More about Django Auth Mixins [[Django Auth Mixins]].
### 1. Background: MVT and Class-Based Views

With **class-based views (CBVs)**, views are classes like `ListView`, `DetailView`, `CreateView`, etc. Instead of using function-based views (FBVs) with decorators (`@login_required`, `@user_passes_test`), CBVs typically use **mixins** to implement reusable view behavior, including authentication/authorization.

---

### 2. What is a Mixin in Django?

A **mixin** is a class meant to be inherited from *in addition to* your main view class to augment it with reusable behavior.

Example structure:

```python
from django.views.generic import ListView

class MyMixin:
    def dispatch(self, request, *args, **kwargs):
        # custom behavior before view logic
        response = super().dispatch(request, *args, **kwargs)
        # custom behavior after view logic
        return response

class MyView(MyMixin, ListView):
    model = MyModel
    template_name = 'my_template.html'
```

For **auth mixins**, the reusable behavior is: “only allow access if certain authentication/authorization rules are met”.

Important note on **inheritance order**:  

```python
class MyView(LoginRequiredMixin, ListView):
    ...
```

The mixin comes **before** the generic view (`ListView`), so that the mixin’s `dispatch` method runs first and can enforce access control [[Multiple Inheritance]].

---

### 3. The Base Class: `AccessMixin`

All the common auth mixins inherit from `django.contrib.auth.mixins.AccessMixin`.

Key attributes and methods:

- `login_url`: where to redirect unauthenticated users. Default: `settings.LOGIN_URL`.
- `redirect_field_name`: query parameter storing the original URL to go back to after login. Default: `"next"`.
- `raise_exception`: if `True`, raise `PermissionDenied` (403) instead of redirecting.
- `handle_no_permission()`: method called when the user fails the test.

You rarely use `AccessMixin` directly, but it’s good to know it underpins the others.

---

### 4. `LoginRequiredMixin`

This mixin ensures the user is **authenticated** (logged in) before accessing the view.

### Basic Usage

```python
from django.contrib.auth.mixins import LoginRequiredMixin
from django.views.generic import ListView
from .models import Post

class PostListView(LoginRequiredMixin, ListView):
    model = Post
    template_name = 'blog/post_list.html'
```

- If user is **not logged in**:
  - Redirected to `LOGIN_URL` (by default `/accounts/login/`) with a `next` parameter (original URL).
- If user **is logged in**:
  - View proceeds as usual (`get`, `post`, etc.).

#### Customizing `login_url` and `redirect_field_name`

```python
class PostListView(LoginRequiredMixin, ListView):
    model = Post
    template_name = 'blog/post_list.html'
    login_url = '/custom-login/'
    redirect_field_name = 'redirect_to'
```

Url after redirect might look like: `/custom-login/?redirect_to=/posts/`.

### Important: MRO and Inheritance Order

Correct:

```python
class PostListView(LoginRequiredMixin, ListView):
    ...
```

Incorrect (mixin last):

```python
class PostListView(ListView, LoginRequiredMixin):  # DON'T DO THIS
    ...
```

If the mixin is placed after `ListView`, its `dispatch` may never be called as intended.
More information about MRO [[Multiple Inheritance]].

---

### 5. `UserPassesTestMixin`

`UserPassesTestMixin` lets you define **arbitrary custom access logic**.

You provide a `test_func` method; only allow access if it returns `True`.

#### Simple Example: Staff-only View

```python
from django.contrib.auth.mixins import UserPassesTestMixin
from django.views.generic import TemplateView

class StaffOnlyView(UserPassesTestMixin, TemplateView):
    template_name = 'staff/dashboard.html'

    def test_func(self):
        return self.request.user.is_staff
```

- If the user fails the test:
  - If `login_url` is set and user is anonymous → redirect to login.
  - If user is authenticated but fails test → by default, redirect to `login_url` or call `handle_no_permission`.
  - If `raise_exception = True`, raise `PermissionDenied` (403).

#### Example: Only Allow Author to Edit Their Own Post

Typical pattern with `UpdateView`:

```python
from django.contrib.auth.mixins import LoginRequiredMixin, UserPassesTestMixin
from django.views.generic import UpdateView
from django.shortcuts import get_object_or_404
from .models import Post

class PostUpdateView(LoginRequiredMixin, UserPassesTestMixin, UpdateView):
    model = Post
    fields = ['title', 'content']
    template_name = 'blog/post_form.html'

    def test_func(self):
        post = self.get_object()
        return post.author == self.request.user
```

Notes:

- We **combine** `LoginRequiredMixin` and `UserPassesTestMixin`.
- Ensure order: `LoginRequiredMixin` before `UserPassesTestMixin` or vice versa is often okay because both override `dispatch` via `AccessMixin`, but convention is:

  ```python
  class View(LoginRequiredMixin, UserPassesTestMixin, UpdateView):
      ...
  ```

  This ensures the user must be authenticated before the custom test is evaluated.

#### Example: Using Groups or Custom Logic

```python
class ManagersOnlyView(UserPassesTestMixin, TemplateView):
    template_name = 'company/manager_dashboard.html'

    def test_func(self):
        user = self.request.user
        return user.is_authenticated and user.groups.filter(name='Managers').exists()
```

If you want to **forbid access entirely** (403) instead of redirecting to login:

```python
class ManagersOnlyView(UserPassesTestMixin, TemplateView):
    template_name = 'company/manager_dashboard.html'
    raise_exception = True # This is what will raise the exception.

    def test_func(self):
        return self.request.user.is_authenticated and \
               self.request.user.groups.filter(name='Managers').exists()
```

---

### 6. `PermissionRequiredMixin`

This mixin enforces Django’s **model-level permissions**, e.g. `add_post`, `change_post`, `delete_post`, `view_post`.

Usage is similar to the decorator `@permission_required`.

### Example: Require a Single Permission

```python
from django.contrib.auth.mixins import PermissionRequiredMixin
from django.views.generic import ListView
from .models import Post

class PostAdminListView(PermissionRequiredMixin, ListView):
    model = Post
    template_name = 'blog/admin_post_list.html'
    permission_required = 'blog.view_post'
```

- `permission_required` can be a string or a list/tuple of strings.
- Permission codename format: `'<app_label>.<permission_codename>'`.

### Example: Multiple Permissions

```python
class PostAdminUpdateView(PermissionRequiredMixin, UpdateView):
    model = Post
    fields = ['title', 'content']
    template_name = 'blog/admin_post_form.html'
    permission_required = ('blog.change_post', 'blog.view_post')
```

You can control whether **all** or **any** permissions are needed:

```python
class PostAdminUpdateView(PermissionRequiredMixin, UpdateView):
    model = Post
    fields = ['title', 'content']
    template_name = 'blog/admin_post_form.html'
    permission_required = ('blog.change_post', 'blog.view_post')
    raise_exception = True  # send 403 instead of redirect

    def has_permission(self):
        # override default behavior if needed
        perms = self.get_permission_required()
        user = self.request.user
        # e.g., require ANY of the listed perms instead of ALL
        return any(user.has_perm(perm) for perm in perms)
```

By default, `PermissionRequiredMixin` requires **all** specified permissions.

---

### 7. Comparisons to Function-Based View Decorators

FBVs traditionally use decorators:

```python
from django.contrib.auth.decorators import login_required, user_passes_test

@login_required
def my_view(request):
    ...

@user_passes_test(lambda u: u.is_staff)
def staff_view(request):
    ...
```

The CBV equivalents using mixins:

- `login_required` → `LoginRequiredMixin`
- `user_passes_test` → `UserPassesTestMixin`
- `permission_required` → `PermissionRequiredMixin`

They do conceptually the same thing, but are integrated via **inheritance** and `dispatch`, not function wrapping.

---

### 8. How Mixins Tie into Dispatch

All these mixins work by overriding the `dispatch()` method of the view:

```python
class AccessMixin:
    def dispatch(self, request, *args, **kwargs):
        # check permission
        if not self.has_permission():
            return self.handle_no_permission()
        return super().dispatch(request, *args, **kwargs)
```

In reality, each mixin customizes `has_permission()` logic:

- `LoginRequiredMixin` → checks `request.user.is_authenticated`
- `UserPassesTestMixin` → calls your `test_func()`
- `PermissionRequiredMixin` → checks `user.has_perm(...)`

This ensures the access check happens **before** `get()`, `post()`, etc.

---

### 9. Practical Examples

#### 9.1. Blog CRUD with Different Access Levels

**List and detail** (anyone can see, but creation/editing restricted).

```python
# views.py
from django.views.generic import ListView, DetailView, CreateView, UpdateView, DeleteView
from django.contrib.auth.mixins import LoginRequiredMixin, UserPassesTestMixin
from django.urls import reverse_lazy
from .models import Post

class PostListView(ListView):
    model = Post
    template_name = 'blog/post_list.html'

class PostDetailView(DetailView):
    model = Post
    template_name = 'blog/post_detail.html'


class PostCreateView(LoginRequiredMixin, CreateView):
    model = Post
    fields = ['title', 'content']
    template_name = 'blog/post_form.html'

    def form_valid(self, form):
        form.instance.author = self.request.user
        return super().form_valid(form)


class PostUpdateView(LoginRequiredMixin, UserPassesTestMixin, UpdateView):
    model = Post
    fields = ['title', 'content']
    template_name = 'blog/post_form.html'

    def test_func(self):
        post = self.get_object()
        return post.author == self.request.user


class PostDeleteView(LoginRequiredMixin, UserPassesTestMixin, DeleteView):
    model = Post
    template_name = 'blog/post_confirm_delete.html'
    success_url = reverse_lazy('post-list')

    def test_func(self):
        post = self.get_object()
        return post.author == self.request.user
```

This is a very common pattern in real projects.

#### 9.2. Admin-only List Using `PermissionRequiredMixin`

```python
from django.contrib.auth.mixins import PermissionRequiredMixin
from django.views.generic import ListView
from .models import UserProfile

class UserProfileListAdminView(PermissionRequiredMixin, ListView):
    model = UserProfile
    template_name = 'accounts/admin_profile_list.html'
    permission_required = 'accounts.view_userprofile'
    raise_exception = True  # 403 instead of redirect if logged-in but unauthorized
```

---

### 10. Best Practices and Common Pitfalls

1. **Place mixins first in the inheritance list**  
   `class MyView(LoginRequiredMixin, ListView): ...`  

2. **Use `LoginRequiredMixin` together with other mixins**  
   For custom tests or permissions, do not rely solely on `UserPassesTestMixin`; enforce authentication as well unless you explicitly want to test anonymous users.

3. **Decide between redirect vs 403**  
   - For not-logged-in users: redirect to login makes sense.
   - For logged-in users lacking permission: often better UX/security to return 403 (set `raise_exception = True`) or redirect with a message.

4. **Keep permission logic close to the view**  
   - For complex access rules, implement them in `test_func()` or override `has_permission()`.
   - Avoid scattering complex logic in templates or URL configs.

5. **Use Django’s built-in permission system**  
   `PermissionRequiredMixin` is powerful when you leverage Django’s `auth_permission` system, especially in admin-heavy or enterprise apps.

6. **Unit test your access control**  
   - Write tests for authenticated vs anonymous, authorized vs unauthorized users.
   - This is where subtle bugs (e.g., wrong inheritance order, incorrect condition) show up.
