# TrenchBoot Website

The TrenchBoot website has been converted over to
[Material for MkDocs](https://squidfunk.github.io/mkdocs-material/). The
content for the original site is still available under the
[old-site-archive](https://github.com/TrenchBoot/TrenchBoot.github.io/tree/old-site-archive).

## Local preview

First prepare virtual environment and install dependencies

```shell
virtualenv -p $(which python3) venv
source venv/bin/activate
pip install -r requirements.txt
```

If you had already done this step then you only need to activate virtual
environment

```shell
source venv/bin/activate
```

To start local server use

```shell
mkdocs serve
```

By default, it will host a local copy of documentation at:
`http://127.0.0.1:8000/`.

If the following error occurs `OSError: [Errno 98] Address already in use`, try
using a different address by running the command `mkdocs serve -a
localhost:12345` (the number is random).

It is crucial at this point to verify that the pages you have changed
render correctly as HTML in local preview.
