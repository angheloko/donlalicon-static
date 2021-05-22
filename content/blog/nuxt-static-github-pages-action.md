---
title: Deploying Nuxt Static Site to Github Pages with GitHub Actions
cover:
  image: https://firebasestorage.googleapis.com/v0/b/donlalicon.appspot.com/o/github-pages.jpg?alt=media&token=59f99fcd-79d4-4e33-a0e6-113fb9434c1c
  thumb: https://firebasestorage.googleapis.com/v0/b/donlalicon.appspot.com/o/github-pages.jpg?alt=media&token=59f99fcd-79d4-4e33-a0e6-113fb9434c1c
createdAt: 2021-05-22
updatedAt: 2021-05-22
---

In a previous article, I talked about [deploying a static Nuxt app to Firebase Hosting](/blog/nuxt-static-firebase-hosting-github-actions). In this article, I'll be sharing about how to deploy a static Nuxt site to GitHub Pages.

## Create a dedicated branch for GitHub Pages

You'll basically create the branch that will be used by GitHub to serve your site. This should **only** contain the build artifact, or the files that are generated if you run `yarn generate`.

This step is optional if you'll be using the [GitHub actions workflow below](#github-actions) to deploy your site. The workflow will automatically create the branch, if needed.

## Configure Nuxt

Make sure that your `nuxt.config.js` contains the following:

```js
export default {
  target: 'static',
  router: {
    base: '/<repository-name>/'
  }
}
```

Where, the `target` property is set to `static` and `repository-name` is your GitHub repository name.

By default, the [router's base property](https://nuxtjs.org/docs/2.x/configuration-glossary/configuration-router/) is `/`. You'll need to override it since project sites are in `https://<username>.github.io/<repository-name>` format.

Doing this ensures that our assets and links are built correctly with respect to the base URL.

## GitHub Actions

Create the `.github/workflows` directory and save the YAML code below inside the directory. You can name the file anything you want as long as it's a `.yml` type (e.g. `gh-pages.yml`).

```yaml[.github/workflows/gh-pages.yml]
name: Deploy to GitHub Pages on merge
'on':
  push:
    branches:
      - master
jobs:
  build_and_deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - run: yarn install --frozen-lockfile && yarn generate
      - uses: peaceiris/actions-gh-pages@v3
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          publish_dir: ./dist
```

The workflow basically involves the following steps:

1. Checkout the master branch.
2. Generate the build artifact.
3. Push the files located in the `dist` directory to the `gh-pages` branch.

By default, the files will be pushed to the `gh-pages` branch. You can change the branch by using the `publish_branch` property.

See [GitHub Actions for GitHub Pages Options](https://github.com/peaceiris/actions-gh-pages#options) for more options.

Commit the file and push your changes. Every time you push your changes, GitHub actions will apply the workflow and automatically build and deploy the changes to your site.

## Select the branch as the source for GitHub Pages

Go to the Settings of your GitHub repository and scroll down to the **Pages** section. In the **GitHub Pages** settings, select the source branch that you created earlier.

![GitHub Pages settings](https://firebasestorage.googleapis.com/v0/b/donlalicon.appspot.com/o/Screenshot%202021-05-22%20at%2012.00.51%20PM.png?alt=media&token=bb883d11-89dc-4553-a686-878c7bdee32d)
