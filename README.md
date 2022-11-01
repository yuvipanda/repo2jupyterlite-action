# repo2jupyterlite GitHub Action

## How to use

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


