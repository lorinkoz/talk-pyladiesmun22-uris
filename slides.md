class: middle center

.small[⏳ Loading...]

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

name: title
class: middle center

# When 404 becomes 302

PyLadies Munich @ Alasco

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

- We knew some radical changes were needed

--

- We couldn't just get rid of the mess and change everything overnight

--

.right[.green[.emph[Cool URIs don't change]]]

---

class: middle center

When you change a URI on your server, you can never completely tell who will have links to the old URI [...] They might have bookmarked your page. They might have scrawled the URI in the margin of a letter to a friend.

.blue[Tim Berners-Lee &mdash; Cool URIs don't change, 1998].ref[1]

.bottom[.left[
.footnote[.ref[1] https://www.w3.org/Provider/Style/URI.html.en]
.footnote[The difference between URI and URL is out of the scope of this talk, but let's just say that URLs are a subset of URIs]
]]

---

class: middle center

### So what should we do? .green[.emph[Design URIs]]

URIs which we will be able to stand by in 2 years, in 20 years, in 200 years

.blue[Also from the World Wide Web guy in the same article].ref[1]

.bottom[.left[
.footnote[.ref[1] https://www.w3.org/Provider/Style/URI.html.en]
]]

---

class: middle center

.huge[🫠]

---

name: the-plan-1
layout: true

## The plan

---

--

1. Design .blue[new] URL structure

--

2. Mount in tandem with .red[old]

--

3. Redirect .red[old] to .blue[new]

--

4. Retire .red[old] URLs (eventually) 🤔

--

.right-column[![Converging railroad tracks](images/tracks.jpeg)]

---

template: the-plan-1
layout: true

|            |                                                              |
| ---------- | ------------------------------------------------------------ |
| .red[old]  | `/estimating/cost_element_budgets/project/1/budget_history/` |
| .blue[new] | `/costs/project/1/budget/history/`                           |

---

--

Phase 1: Design .blue[new]

Phase 2: .red[old] and .blue[new] both respond the same

Phase 3: .red[old] redirects to .blue[new]

Phase 4: .red[old] is removed, only .blue[new] remains

---

layout: false

## Prerequisites

--

🤓 Because we wanted to solve by code...

--

.left-column[

#### Framework

- ✅ Named URLs
- ✅ Namespaced URLs

]

--

.right-column[

#### Our code

- 😒 Keep names in sync
- ✅ Move old URLs to specific namespace

]

---

class: middle center

# ⚙️ Turning 404s into 302s

---

name: code-warning
class: middle center

![Sign with a warning about code ahead](images/code-ahead.png)

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
            # Because it's humans keeping things in sync
            logger.warning("🚧")

    return None

```

---

## Do we want to handle?

--

```python
def we_want_to_handle(request):
    return (
        response.status_code not in [301, 302]
        and request.method == "GET"  # Wait, why?
    )
```

--

.box[💡 Did you know there is also .green[307] and .green[308]?]

---

## Is old URL?

--

```python
def is_old_url(request):
    resolver_match = request.resolver_match

    # Having namespaced the old URLs comes in handy now
    return "old_namespace" in resolver_match.app_names
```

---

## Find new URL!

--

```python
def find_new_url(request):
    resolver_match = request.resolver_match

    # Ah yes, view comes with all namespaces prefixed
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

class: middle

.left-column-66[![Multiple railroad tracks](images/tracks2.gif)]

--

.right-column-33[

- Adjusting the wording (e.g. replace "amendment" with "change order")
- Integrating multiple vanilla Django pages into single page with frontend component.

]

---

## The plan

--

- Big redesign was already done, it was a matter of ajusting here and there

--

- No need to mount parallel structures again

--

#### .right[.green[Can't we just rely on smart path aliasing?]]

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

template: code-warning

---

layout: true

## Here we go again: "path with old"

---

```python
def path_with_old(route, view, kwargs=None, name=None, `*, old=None`):
    paths = [path(route, view, kwargs, name)]

    if name and old:
        # Redirect all "old" to "route" using "name"
        ...

    return paths[0]
```

---

```python
def path_with_old(route, view, kwargs=None, name=None, `*, old=None`):
    paths = [path(route, view, kwargs, name)]

    if name and old:
        # Redirect all "old" to "route" using "name"
        ...

    return path("", include(paths)) if len(paths) > 1 else paths[0]
```

---

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

layout: false

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

#### After that we just had to start making changes and enhancing the paths

--

## And we never had to make any changes in URLs ever again

--

#### (guess what 🙃)

---

## But let's just stop here

--

#### Where to find me?

.left-column-66[

|         |                                                    |
| ------- | -------------------------------------------------- |
| Twitter | [@lorinkoz](https://twitter.com/lorinkoz)          |
| GitHub  | [github.com/lorinkoz](https://github.com/lorinkoz) |
| Email   | [lorinkoz@gmail.com](mailto:lorinkoz@gmail.com)    |

.right[Slides are here 👉]
.right[.small[(and you should never get a 404 from this link)]]
]

.right-column-33[![Slides QR Code](images/slides-qr.png)]

---

template: title
