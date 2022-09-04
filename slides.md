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
.right-column-33[![Myself](images/lorinkoz.png)]

---

## The problem

.left-column-33[![Post with directions](images/directions.jpeg)]

--

- Moving fast

--

- Redefining the answer to .blue["what are we building?"]

--

- Our URLs didn't reflect the experience of the product anymore

--

- Tim Berners-Lee (inventor of the World Wide Web)

---

class: middle center

When you change a URI on your server, you can never completely tell who will have links to the old URI [...] They might have bookmarked your page. They might have scrawled the URI in the margin of a letter to a friend.

.blue[Tim Berners-Lee &mdash; Cool URIs don't change, 1998]

---

## The ideal solution

--

- Define .blue[new] URL structure

--

- Mount in tandem with .red[old]

--

- Redirect .red[old] to .blue[new]

--

- Retire .red[old] URLs (eventually) 🤔

--

.right-column[![Converging railroad tracks](images/tracks.jpeg)]

---

## Prerequisites

--

.left-column[

#### Framework

- ✅ Named URLs
- ✅ Namespaced URLs

]

--

.right-column[

#### Our code

- 😒 Keep names
- ✅ Move old URLs to specific namespace

]

---

class: middle center

# ⚙️ Turning 404s into 302s

---

## Fun fact: Redirects

|                           | Temporary         | Permanent         |
| ------------------------- | ----------------- | ----------------- |
| Method change allowed     | .center[3️⃣ 0️⃣ 2️⃣] | .center[3️⃣ 0️⃣ 1️⃣] |
| Method change NOT allowed | .center[3️⃣ 0️⃣ 7️⃣] | .center[3️⃣ 0️⃣ 8️⃣] |

---

## A middleware!

--

```python
def redirect_middleware(get_response):

    def middleware(request):
        response = get_response(request)

        if go_to_new := `should_go_to_new`(request)
            return redirect(go_to_new, permanent=True)

        return response

    return middleware
```

---

## A "should go to new" function!

--

```python
def should_go_to_new(request):
    if `we_want_to_handle`(request) and `is_old_url`(request):
        try:
            return `find_new_url`(request)
        except NoReverseMatch:
            logger.warning("🚧")

    return None

```

---

## Do we want to handle?

--

```python
def we_want_to_handle(request):
    return (
        request.method == "GET"
        and response.status_code not in [301, 302]
    )
```

--

.box[🤔 What to do with `POST` and the others?]

---

## Is old URL?

--

```python
def is_old_url(request):
    resolver_match = request.resolver_match
    return "old_namespace" in resolver_match.app_names
```

---

## Find new URL!

--

```python
def find_new_url(request):
    resolver_match = request.resolver_match

    view_name = resolver_match.view_name.split(":")[-1]

    return reverse(
        view_name,
        args=resolver_match.args,
        kwargs=resolver_match.kwargs
    )
```

--

.box[🤔 What about query parameters?]

---

class: middle center

#### After that we just had to install the new ✨ middleware

--

## And we never had to make any changes in URLs ever again

--

#### (until one month later)

---

class: middle center

![Multiple railroad tracks](images/tracks2.gif)

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
path_with_old(
    "users/<int:pk>/",
    UserDetailUpdateView.as_view(),
    name="user_detail_update",
    old=[
        "users/<int:pk>/invite/",
        "users/<int:pk>/toggle/",
        "users/<int:pk>/revoke/",
        "users/<int:pk>/delete/",
    ],
)
```

---

## Here we go again: "path with old"

```python
def path_with_old(route, view, kwargs=None, name=None, *, old=None):
    paths = [path(route, view, kwargs, name)]

    if name and old:
        redirect_view = `get_redirect_view`(name)

        for idx, old_path in enumerate(old):
            redirect_path = path(
                old_path,
                redirect_view,
                kwargs,
                f"{name}__{idx}"
            )
            paths.append(redirect_path)

    return path("", include(paths)) if len(paths) > 1 else paths[0]
```

---

## A redirect view on-the-fly

```python
def get_redirect_view(name):

    def redirect_view_on_the_fly(request, *args, **kwargs):
        resolver_match = request.resolver_match
        full_name = ":".join([*resolver_match.namespaces, name])
        return redirect(full_name, *args, **kwargs, permanent=True)

    return redirect_view_on_the_fly
```

---

class: middle center

#### After that we just had to start enhancing paths

--

## And we _really_ never had to make any changes in URLs ever again

--

#### (guess what 🙃)

---

## But let's just stop here

#### Where to find me?

.left-column-66[

|         |                                                    |
| ------- | -------------------------------------------------- |
| Twitter | [@lorinkoz](https://twitter.com/lorinkoz)          |
| GitHub  | [github.com/lorinkoz](https://github.com/lorinkoz) |
| Email   | [lorinkoz@gmail.com](mailto:lorinkoz@gmail.com)    |

.right[Slides are here 👉]
]

.right-column-33[![Slides QR Code](images/slides-qr.png)]

---

template: title
