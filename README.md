# ARC GNU Toolchain Documentation

The documentation is powered by [MkDocs](https://www.mkdocs.org/)
Markdown-based generator.

`main` branch contains the latest documentation updates. After the release a
separate branch must be created for it. Thus, `main` always contains the latest
updates and a release branch (for example, `2025.09`) contains only a cutoff for
a particular release.

## Prepare Build Environment

Create a Python virtual environment, install dependencies and activate
the environment:

```shell
python3 -m venv .venv
source .venv/bin/activate
pip3 install -r requirements.txt
```

## MkDocs Commands

| Command        | Description                                                                      |
|----------------|----------------------------------------------------------------------------------|
| `mkdocs build` | Build the documentation. Generated pages are stored in `site` subdirectory.      |
| `mkdocs serve` | Start HTTP-server for the documentation and serve it on <http://127.0.0.1:8000>. |

## Deploying the Documentation

[Mike](https://github.com/jimporter/mike) utility is used for deploying the
documentation. It allows to deploy different version of the documentation
at the same time.

Use `mike deploy` command to build and commit the documentation from the
current branch to `gh-pages` branch:

```shell
mike deploy 2025.09
```

If you are ready to push it to the remote repository, then also use `--push`
command:

```shell
mike deploy --push 2025.09
```

List all deployed documentation versions:

```shell
$ mike list
2025.09
2025.06
2024.12 [latest]
2024.06
2023.09
```

Set the default version:

```shell
mike set-default 2025.09
```

## Useful Resources

* [MkDocs page](https://www.mkdocs.org/)
* [Mike page](https://github.com/jimporter/mike)
* [MkDocs Material theme page](https://squidfunk.github.io/mkdocs-material/)
* [Markdown syntax guide](https://daringfireball.net/projects/markdown/syntax)
* [A list of languages for Pygments syntax highlighter](https://pygments.org/languages/)
