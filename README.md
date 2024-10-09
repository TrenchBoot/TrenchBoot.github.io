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

It is also important to read console output when starting mkdocs as it shows
useful information such as existing files that are not added to `nav` or missing
files.

* console output when there are documents not added to `nav`

    ```text
    INFO    -  The following pages exist in the docs directory, but are not included
            in the "nav" configuration:
                - specifications/Secure_Launch.md
    ```

* console output when files added to `nav` can't be found

    ```text
    WARNING -  A reference to 'specifications/no_such_file.md' is included in the
            'nav' configuration, which is not found in the documentation files.
    ```

## Contributing

Prepare and enable virtual environment as described in
[Local preview](#local-preview) and after that install pre-commit hooks.

```shell
$ pre-commit install
pre-commit installed at .git/hooks/pre-commit
pre-commit installed at .git/hooks/commit-msg
```
