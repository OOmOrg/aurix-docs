# Aurix External Documentation

This repository contains the external documentation for [aurix.com.sg](https://aurix.com.sg).

The site is built using [Zensical](https://zensical.org/docs/get-started/), a modern static site generator for project documentation.

## Getting Started

To build or preview the documentation locally, ensure you have [Python](https://www.python.org/) installed and then set up a virtual environment:

```sh
python3 -m venv myenv
source myenv/bin/activate
pip install zensical
```

You can then build the site with:

```sh
zensical build --clean
```

Run Locally 
```sh
zensical serve
```

For more details, see the [Zensical documentation](https://zensical.org/docs/get-started/).
