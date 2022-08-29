name: title
class: middle center

# When 404 becomes 302

PyLadies Munich @ Alasco

---

## 👋 Hola

.left-column-66[

#### Lorenzo Peña

- Backend Engineer at Alasco
- 13 years of Python + Django
- Proud father of this little lady

]
.right-column-33[![Foto mía](images/lorinkoz.png)]

---

class: middle center

## Cool URIs don't change

---

## Prerequisites

- Named URLs
- Namespaced URLs

---

## A middleware!

```python
def redirect_middleware(get_response):

    def middleware(request):
        response = get_response(request)

        if go_here := should_go_here(request)
            return redirect(go_here)

        return response

    return middleware
```

---

name: should-go-here

## A "should go here" function!

```python
def should_go_here(request):
    if we_want_to_handle(request) and is_old_url(request):
        try:
            return find_new_url(request)
        except NoReverseMatch:
            logger.warning("🚧")

    return None

```

---

## Do we want to handle?

```python
def we_want_to_handle(request):
    return (
        request.method == "GET"
        and response.status_code not in [301, 302]
    )
```

.box[🤔 What to do with `POST` and the others?]

---

## Is old URL?

```python
def is_old_url(request):
    resolver_match = request.resolver_match
    return "old_namespace" in resolver_match.app_names
```

---

## Find new URL!

```python
def find_new_url(request):
    resolver_match = request.resolver_match

    return reverse(
        resolver_match.view_name.split(":")[-1],
        args=resolver_match.args,
        kwargs=resolver_match.kwargs
    )
```

.box[🤔 What about query parameters?]

---

class: middle center

## And we never had to change any URLs again

#### ...until one month later

---

## Vanilla path

```python
path(
    "users/<int:pk>/",
    UserDetailUpdateView.as_view(),
    name="user_detail_update",
)
```

---

## Cranberry path

```python
path_with_old_paths(
    "users/<int:pk>/",
    UserDetailUpdateView.as_view(),
    name="user_detail_update",
    old_paths=[
        "users/<int:pk>/invite/",
        "users/<int:pk>/toggle/",
        "users/<int:pk>/revoke/",
        "users/<int:pk>/delete/",
    ],
)
```

---

## That's a wrap

#### You can find me here:

|         |                                                    |
| ------- | -------------------------------------------------- |
| Twitter | [@lorinkoz](https://twitter.com/lorinkoz)          |
| GitHub  | [github.com/lorinkoz](https://github.com/lorinkoz) |
| Correo  | [lorinkoz@gmail.com](mailto:lorinkoz@gmail.com)    |

---

template: title
