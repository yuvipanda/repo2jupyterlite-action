# repo2jupyterlite GitHub Action

## Setting up publishing with GitHub Actions

1. [Enable publishing to GitHub Pages with GitHub Actions](https://docs.github.com/en/pages/getting-started-with-github-pages/configuring-a-publishing-source-for-your-github-pages-site#publishing-with-a-custom-github-actions-workflow)
   for your repository.
2. Create a `.github/workflows/publish.yaml` file in your GitHub repository with the following
   contents.


```yaml
name: Build and Publish jupyterlite page to GitHub Pages

on:
  push:
    branches:
      - main
  pull_request:
    branches:
      - '*'

jobs:
  build:
    runs-on: ubuntu-latest
    defaults:
      run:
        shell: bash -l {0}
    steps:
      - name: Checkout
        uses: actions/checkout@v3
      - name: Build repo2jupyterlite
        uses: yuvipanda/repo2jupyterlite-action@main
      - name: Upload generated site
        uses: actions/upload-pages-artifact@v1
        with:
          path: ./dist

  publish:
    needs: build
    if: github.ref == 'refs/heads/main'
    permissions:
      pages: write
      id-token: write

    environment:
      name: github-pages
      url: ${{ steps.publish.outputs.page_url }}

    runs-on: ubuntu-latest
    steps:
      - name: Publish to GitHub Pages
        id: publish
        uses: actions/deploy-pages@v1
```

3. After you commit this workflow, go to the *Actions* tab in your repo and watch to see
   if the build and publish succeeded. If it did succeed, the URL to your published
   JupyterLite instance would be shown next to the `Publish` step. Usually this is of
   the form `https://<your-github-username>.github.io/<repo-name>`.


## Install additional packages

`repo2jupyterlite` supports using [mamba with emscriptenforge](https://blog.jupyter.org/mamba-meets-jupyterlite-88ef49ac4dc8)
to install additional packages that are immediately available to your users when they start the JupyterLite
instance. Not all packages are available, but the following are:

1. All packages in [conda-forge](https://conda-forge.org/) that have *no C or compiled*
   dependencies (aka `noarch`). This is the majority of packages!
2. Special packages with [emscriptenforge recipes](https://github.com/emscripten-forge/recipes/tree/main/recipes/recipes_emscripten)

Eventually we will support traditional `environment.yml` files, but for now, you have to create a
`jupyter_lite_config.json` file. Add the following contents to it:

```json
{
    "XeusPythonEnv": {
        "packages": [
            "package-1",
            "package-2==<some-version>"
        ]
    }
}
```

If you commit the `jupyter_lite_config.json` file to your repo, the action will automatically use
`mamba` to install these packages and make them available to the statically built output site. Note
that you **must** select the **XPython** kernel, **not** the *Pyodide* kernel to have access to these
extra packages.