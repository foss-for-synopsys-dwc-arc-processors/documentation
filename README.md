# ARC GNU Toolchain Documentation

## Using MkDocs

Prepare an environment:

```shell
$ python3 -m venv .venv
$ source .venv/bin/activate
$ pip3 install -r requirements.txt
```

Build the documentation:

```shell
$ source .venv/bin/activate
$ mkdocs build
```

Run test server:

```shell
$ source .venv/bin/activate
$ mkdocs serve
```

Deploy the documentation:

```shell
$ source .venv/bin/activate
$ mkdocs gh-deploy
```

## Useful Resources

* [MkDocs page](https://www.mkdocs.org/)
* [MkDocs Material theme page](https://squidfunk.github.io/mkdocs-material/)
* [Markdown syntax guide](https://daringfireball.net/projects/markdown/syntax)
* [A list of languages for Pygments syntax highlighter](https://pygments.org/languages/)
