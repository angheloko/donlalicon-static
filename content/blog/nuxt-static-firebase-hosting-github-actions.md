---
title: Deploy Nuxt on Firebase Hosting with GitHub Actions
createdAt: 2021-02-23
subtitle: Automating building and deploying to Firebase Hosting with GitHub Actions.
cover:
  image: https://firebasestorage.googleapis.com/v0/b/donlalicon.appspot.com/o/pexels-digital-buggu-171198.jpg?alt=media&token=e06bf63b-bf9e-4870-ad6c-51d99e90c8f9
  alt: Photo by Digital Buggu from Pexels
  caption: <a href="https://www.pexels.com/photo/colorful-toothed-wheels-171198/?utm_content=attributionCopyText&utm_medium=referral&utm_source=pexels">Photo by Digital Buggu from Pexels</a>
  thumb: https://firebasestorage.googleapis.com/v0/b/donlalicon.appspot.com/o/pexels-digital-buggu-171198-thumb.jpg?alt=media&token=6cee0015-1d7b-4143-9ee5-95577ed6d631
tags:
  - Nuxt.js
  - Firebase Hosting
  - GitHub Actions
---

Nuxt.js is easily one of the best static site generators right now and deploying it to any static hosting site is effortless.

## Generate your static site

To deploy, you'll need to generate your static website first: 

```shell
yarn generate
```

If you haven't done so, make sure that the `target` property in your `nuxt.config.js` is set to `static`:

```js[nuxt.config.js]
export default {
  // Target: https://go.nuxtjs.dev/config-target
  target: 'static'
}
```

## Deploying using the Firebase CLI

You'll need to [set up a Firebase project](https://firebase.google.com/docs/web/setup) and [install the Firebase CLI](https://firebase.google.com/docs/cli).

Once you've [added and initialized Firebase in your project](https://firebase.google.com/docs/cli#initialize_a_firebase_project), you should have a file called `firebase.json` in your application root.

```json[firebase.json]
{
  "hosting": {
    "public": "public",
    "ignore": [
      "firebase.json",
      "**/.*",
      "**/node_modules/**"
    ]
  }
}
```

Since Nuxt.js generates its distribution files in the [`dist` directory by default](https://nuxtjs.org/docs/2.x/directory-structure/dist/), you should also update the `public` property in your `firebase.json` to `dist`:

```json[firebase.json]
{
  "hosting": {
    "public": "dist",
    "ignore": [
      "firebase.json",
      "**/.*",
      "**/node_modules/**"
    ]
  }
}
```

To deploy your static website to Firebase hosting, you can now simply run:

```shell
firebase deploy --only hosting
```

## Automate deployment with GitHub Actions

If you're using GitHub, you can use [GitHub Actions](https://github.com/features/actions) to automatically [generate and deploy your site to Firebase Hosting](https://firebase.google.com/docs/hosting/github-integration) whenever a pull request is merged to your default branch.

To automatically generate the GitHub Actions configuration files for your site, run the following:

```shell
firebase init hosting:github
```

The command above will generate 2 files:
- `.github/workflows/firebase-hosting-merge.yml` - is triggered whenever a branch or pull request is merged to your default branch. The application will be deployed to your production site.
- `.github/workflows/firebase-hosting-pull-request.yml` - is triggered whenever a pull request is created. The application will be deployed to a [beta channel](https://firebase.google.com/docs/hosting/test-preview-deploy).

### Using yarn instead of npm

By default, the workflow configuration will be using `npm`. If you prefer to use `yarn`, you'll need to replace the command that builds your distribution files.

Replace the `npm` commands:

```shell
npm ci && npm run generate
```

To `yarn` commands:

```shell
yarn install --frozen-lockfile && yarn generate
```

### Replacing sensitive data

You can use [GitHub secrets](https://docs.github.com/en/actions/reference/encrypted-secrets) to store sensitive information such as API keys and have them automatically inserted when building your app with GitHub actions. For example, if you want to replace an API key, you can simply add a step in your workflow that replaces a placeholder with the actual value:

```yaml
- run: sed -i "s/API_KEY/${API_KEY}/" app.html
  env:
    API_KEY: ${{ secrets.FIREBASE_API_KEY }}
```

In the step above, we use `sed` to replace instances of `${API_KEY}` with the actual API key.

## Real-world example

If you're looking for an actual working example, look no further! This website is using [GitHub actions](https://github.com/angheloko/donlalicon-static/tree/master/.github/workflows) to automatically build and deploy itself.


