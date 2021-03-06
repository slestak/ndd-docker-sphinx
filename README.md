# NDD Docker Sphinx

1. [Introduction](#markdown-header-introduction)
1. [Installation](#markdown-header-installation)
    1. [From source](#markdown-header-from-source)
    1. [From Docker Hub](#markdown-header-from-docker-hub)
1. [Usage](#markdown-header-usage)
    1. [Initialisation](#markdown-header-initialisation)
    1. [Interactive](#markdown-header-interactive)
    1. [Non interactive](#markdown-header-non-interactive)
1. [Configuration](#markdown-header-configuration)
    1. [Extensions](#markdown-header-extensions)
    1. [Install an extension](#markdown-header-install-an-extension)
1. [Custom "extensions"](#custom-extensions)
    1. [Git extension"](#git-extension)
1. [Limitations](#markdown-header-limitations)



## Introduction

A Docker image for the [Sphinx documentation](http://sphinx-doc.org) builder.

The image is based upon the official [python:3.6](https://hub.docker.com/_/python/).

Besides the Sphinx documentation builder ([sphinx-doc](http://sphinx-doc.org)), this image contains:

- [alabaster](https://pypi.python.org/pypi/alabaster)
- [gitpython](https://pypi.python.org/pypi/gitpython)
- [guzzle-sphinx-theme](https://pypi.python.org/pypi/guzzle_sphinx_theme)
- [livereload](https://pypi.python.org/pypi/livereload)
- [sphinx-autobuild](https://pypi.org/project/sphinx-autobuild)
- [sphinx-bootstrap-theme](https://pypi.python.org/pypi/sphinx-bootstrap-theme)
- [sphinx-prompt](https://pypi.python.org/pypi/sphinx-prompt)
- [sphinx-rtd-theme](https://pypi.python.org/pypi/sphinx_rtd_theme)
- [sphinxcontrib-actdiag](https://pypi.python.org/pypi/sphinxcontrib-actdiag)
- [sphinxcontrib-blockdiag](https://pypi.python.org/pypi/sphinxcontrib-blockdiag)
- [sphinxcontrib-excel-table](https://pypi.python.org/pypi/sphinxcontrib-excel-table)
- [sphinxcontrib-fulltoc](https://pypi.org/project/sphinxcontrib-fulltoc)
- [sphinxcontrib-googleanalytics](https://pypi.python.org/pypi/sphinxcontrib-googleanalytics)
- [sphinxcontrib-googlechart](https://pypi.python.org/pypi/sphinxcontrib-googlechart)
- [sphinxcontrib-googlemaps](https://pypi.python.org/pypi/sphinxcontrib-googlemaps)
- [sphinxcontrib-nwdiag](https://pypi.python.org/pypi/sphinxcontrib-nwdiag)
- [sphinxcontrib-plantuml](https://pypi.python.org/pypi/sphinxcontrib-plantuml)
- [sphinxcontrib-seqdiag](https://pypi.python.org/pypi/sphinxcontrib-seqdiag)

The versioning scheme of this Docker image is `<SPHINX_VERSION>-<DOCKER_IMAGE_VERSION>`.
For example, `1.8.1-9` stands for the 9th version of the Docker image using Sphinx 1.8.1.



## Installation

### From source

```shell
git clone git@bitbucket.org:ndd-docker/ndd-docker-sphinx.git
cd ndd-docker-sphinx
docker build -t ddidier/sphinx-doc .
```

### From Docker Hub

```shell
docker pull ddidier/sphinx-doc
```



## Usage

The documentation directory on the host `<HOST_DATA_DIR>` must be mounted as a volume under `/doc` in the container.
Use `-v <HOST_DATA_DIR>:/doc` to use a specific documentation directory or `-v $(pwd):/doc` to use the current directory as the documentation directory.

Sphinx will be executed inside the container by the `sphinx-doc` user which is created by the Docker entry point.
You **must** pass to the container the environment variable `USER_ID` set to the UID of the user the files will belong to.
This is the ``-e USER_ID=$UID `` part in the examples of this documentation.

### Initialisation

Sphinx provides the [`sphinx-quickstart`](http://sphinx-doc.org/invocation.html) script to create a skeleton of the documentation directory.
You should however use the custom tailored `sphinx-init` script which:

- calls `sphinx-quickstart`
- customizes the `Makefile` (`livehtml` target)
- customizes the configuration file `conf.py` (offcial and custom extensions, markdown support, theme)

**The directory `<HOST_DATA_DIR>` must already exist, otherwise the script will fail!**

```shell
docker run -it -v <HOST_DATA_DIR>:/doc -e USER_ID=$UID ddidier/sphinx-doc sphinx-init
```

All arguments accepted by [`sphinx-quickstart`](http://sphinx-doc.org/invocation.html) are passed to `sphinx-init`.
For example, you may want to pass the project name on the command line:

```shell
docker run -it -v <HOST_DATA_DIR>:/doc -e USER_ID=$UID ddidier/sphinx-doc sphinx-init --project my-documentation
```

### Interactive

The so-called *interactive mode* is when you issue commands from inside the container.

```shell
docker run -it -v <HOST_DATA_DIR>:/doc -e USER_ID=$UID ddidier/sphinx-doc
```

You should now be in the `/doc` directory, otherwise just `cd` to `/doc`.

To see all the official targets, call `make help`.

To create a PDF document, call `make latexpdf`.

To create HTML documents, call `make html`.

### Non interactive

The so-called *non-interactive mode* is when you issue commands from the host directly.

To see all the official targets, call:

```shell
docker run -i -v <HOST_DATA_DIR>:/doc -e USER_ID=$UID ddidier/sphinx-doc make help
```

To create a PDF document, call `make latexpdf`.

```shell
docker run -i -v <HOST_DATA_DIR>:/doc -e USER_ID=$UID ddidier/sphinx-doc make latexpdf
```

To create HTML documents, call `make html`.

```shell
docker run -i -v <HOST_DATA_DIR>:/doc -e USER_ID=$UID ddidier/sphinx-doc make html
```

To create HTML documents and watch for changes, call `make livehtml`:

```shell
docker run -it -v <HOST_DATA_DIR>:/doc -p 8000:8000 -e USER_ID=$UID ddidier/sphinx-doc make livehtml
#                                         ^^^^
#                                         open your browser at http://localhost:8000/
```

To trigger a full build while in watch mode, issue from the `<HOST_DATA_DIR>` folder on the host:

```shell
rm -rf build && touch source/conf.py
```

### Tips & Tricks

If you need the directory `<HOST_DATA_DIR>` to NOT be the root of the documentation, change the `make` directory with `-C`.
Previous commands such as `make target` become `make -C /some/directory/ target`.
Please see the pseudo [Git extension below](#git-extension) for an example.



## Configuration

**Warning: to simplify the `sphinx-init` script, some variables are overriden at the end of the `conf.py` file.**
This is most notably the case of the `extensions` and `html_theme` variables.

### Bundled extensions

This image comes with a number of already bundled extensions.

To enable a bundled extension, simply uncomment the associated line in your `conf.py`.

To disable a bundled extension, simply comment the associated line in your `conf.py`.

### Other extensions

IF you want to use an extension which is not already bundled with this image, you need to:

First extend the image by creating a new `Dockerfile`:

```docker
FROM ddidier/sphinx-doc:latest

RUN pip install 'a-sphinx-extension       == A.B.C' \
                'another-sphinx-extension == X.Y.Z'
```

Then add a line in your `conf.py` referencing the extension:

```python
extensions = [
    ...
    'a.sphinx.extension',
    'another.sphinx.extension',
]
```



## Custom "extensions"

This should be extracted in actual Sphinx extensions...

At the time being, the Python code is stored in the subdirectory `/_python` and is copied when calling `sphinx-init`.

### Git extension

This pseudo extension reads the properties of a Git repository to display in the left navigation panel and in the footer:

- the current time of the build if any file is not committed or untracked, or
- the name of the tag associated with the last commit if it exists, or
- the hash of the last commit

Enable it by uncommenting the following lines in your `conf.py`:

```python
# Must be defined somewhere
html_context = {}

import os.path
source_directory = os.path.dirname(os.path.realpath(__file__))
python_directory = os.path.join(source_directory, '_python')
exec(open(os.path.join(python_directory, 'sphinx-git.py'), 'rb').read())
```

This extension requires access to the `.git` directory:

1. if the documentation is at the same level than the `.git` directory:

    ```shell
    # /my-project
    # ├── .git
    # ├── Makefile
    # ├── build
    # └── source
    #     └── conf.py

    docker run -i -v /my-project:/doc -e USER_ID=$UID ddidier/sphinx-doc make html
    ```

2. if the documentation is not at the same level than the `.git` directory:

    ```shell
    # /my-project
    # ├── .git
    # ├── sources
    # └── documentation
    #     ├── Makefile
    #     ├── build
    #     └── source
    #         └── conf.py

    docker run -i -v /my-project:/doc -e USER_ID=$UID ddidier/sphinx-doc make -C /doc/documentation html
    ```



## Limitations

- PDF generation does not work when including Markdown file using `recommonmark`.
- PDF generation does not take into account Excel tables.
