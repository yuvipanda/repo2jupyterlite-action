# repo2jupyterlite GitHub Action

Publish a repository full of interactive Jupyter Notebooks to GitHub pages
statically with [JupyterLite](https://jupyterlite.readthedocs.io/en/latest/),
allowing visitors to explore your notebooks (and execute them) *entirely in
the browser*, without any backend necessary!

## Setting up publishing with GitHub Actions

1. [Enable publishing to GitHub Pages with GitHub Actions](https://docs.github.com/en/pages/getting-started-with-github-pages/configuring-a-publishing-source-for-your-github-pages-site#publishing-with-a-custom-github-actions-workflow)
   for your repository.
2. Create a `.github/workflows/publish.yaml` file in your GitHub repository with the following
   contents. If using the GitHub Web UI, you can Select 'Add file' -> 'Create new file' from the
   UI when viewing the root directory of your repository, and type in `.github/workflows/publish.yaml`
   into the file name. This will create the appropriate subdirectories if needed too.


    ```yaml
    name: Build and Publish jupyterlite page to GitHub Pages
    on:
      push:
        branches:
        - main  # specify 'master' if that is your main branch
      pull_request:
        branches:
        - '*'

    jobs:
      build:
        runs-on: ubuntu-latest
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
   the form `https://<your-github-username>.github.io/<repo-name>`. This URL can also
   be found if you select "Settings" from your repository page, and then select "Pages"
   from the left sidebar.


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

## Limitations

1. The action will fail if you have any directories starting with `.` in your repository
[Issue](https://github.com/jupyterlite/jupyterlite/issues/624).
2. If you push a change to your repo installing a new package, it might take a few minutes sometimes
   for cache to update on your local machine before you can use that package. Try a different browser,
   and be patient.

## Optional Inputs

The action works by default without any parameters, but you can customize the action
with some optional parameters.

1. `contents_path`: The path containing the contents to be bundled into the JupyterLite.
   Defaults to `.`, which is the current directory.
2. `destination_path`: The path where the generated static files are placed. This is
   what should be uploaded to GitHub Pages. Defaults to `./dist`.
3. `repo2jupyterlite_version`: Version of repo2jupyterlite to use.

   `stable` installs latest version from PyPI, `latest` installs latest version
   from the repo2jupyterlite git repository, any other string installs that specific
   version from PyPI
