---
layout: post
title: How to deploy a React app with Kamal (formerly known as MRSK) & GitHub Action
date: 2024-06-17 10:28 +0530
---
I recently helped Sidecar Learning move their legacy application built with React front-end and Rails back-end migrate from Heroku and AWS to VPS over at DigitalOcean.

This guide is outcome of that migration. React front-end was quite simple and do not have much moving parts so it's quite simple to do this. If you are looking to deploy monolith application then you can read the following posts I have written:

- [Deploy Rails app and Postgres with Kamal(previously MRSK) on single DigitalOcean server](/2023/04/05/how-to-deploy-rails-app-and-postgres-with-mrsk-on-single-server.html)
- [How to deploy multi-environment(staging, production) application using Kamal(previously MRSK)](/2023/04/09/how-to-deploy-multi-environment-staging-production-application-using-mrsk.html)
- [How to deploy a NodeJS application using Kamal (previously MRSK)](/2023/04/10/how-to-deploy-a-nodejs-application-using-mrsk.html)

## Kamal Config

First, let's create a simple `deploy.yml` for Kamal. The configuration is mostly boilerplate. I am using [GHA cache](https://docs.docker.com/build/cache/backends/gha/) to speed up the deploys.

```yaml
# config/deploy.yml

service: react-app
image: username/image-name

servers:
  web:
    - 123.456.789.012

registry:
  username: docker_username
  password:
    - KAMAL_REGISTRY_PASSWORD

env:
  clear:
    FOO: BAR
    NPM_CONFIG_PRODUCTION: false

builder:
  multiarch: false
  cache:
    type: gha
    options: mode=max
```

If you wish to, you can use "registry" cache as well. I have noticed that it's faster by 5-10 seconds but if you are using free Docker Hub account then you only get one free image. That's why I prefer to use "gha" as cache back-end. Kamal only [supports](https://kamal-deploy.org/docs/configuration/builders/#using-multistage-builder-cache) "gha" and "registry". If you would like to use the "registry" cache, use the following builder config:
```yaml
builder:
  multiarch: false
  cache:
    type: registry
    options: mode=max,image-manifest=true,oci-mediatypes=true
```



## Dockerfile

Our app is using quite old version of NodeJS. This is why I like Kamal more than anything. No platform dependency or requirement to fulfil. You create your own environment and Kamal provides thin wrapper around the Docker commands. As simple as it could be.

Here, we first create a build step. Once the build is ready, no need to include all those node_modules in our final application. We keep it as small as possible so that it's easier to perform operations.

In our app, we have a small Express app that serves the static file. That's why I have created a separate `package.runtime.json` file. You can modify that part as per your own need. The next session will explain how the express server works.

```dockerfile
# Dockerfile

FROM node:11.10.1 as build

WORKDIR /app

ENV NODE_ENV="production" \
    NPM_CONFIG_PRODUCTION="false"

COPY . .

RUN npm ci

RUN npm run build

FROM node:11.10.1

WORKDIR /app

ENV NODE_ENV="production" \
    NPM_CONFIG_PRODUCTION="false"

COPY --from=build /app/build ./build
COPY --from=build /app/server.js ./server.js
COPY --from=build /app/package.runtime.json ./package.json

RUN npm install --production

EXPOSE 3000

CMD ["node", "server.js"]
```



## Express Server

We have a tiny express server that serves the static files with appropriate headers. We have configured Cloudflare to provide SSL support and cache the static assets. Only `index.html` file is not cached, and bundled assets will be served by this Express server only once. Then Cloudflare will take the load. All of this is deployed on our $4 DigitalOcean droplet and it is handling our moderately busy app very well.

I have [configured](https://webpack.js.org/guides/caching/) the Webpack to generate bulid with hash, so with every new build, the new hash will rename the file - same as [Propshaft](https://github.com/rails/propshaft/).

I have added a route for Kamal healthcheck as well even though the health-check step has been [removed](https://github.com/basecamp/kamal/pull/740) in Kamal 1.6.0.

```javascript
// server.js

const compression = require('compression');
const express = require('express');
const path = require('path');
const port = process.env.PORT || 3000;
const app = express();

// Use compression middleware
app.use(compression());

// Serve static files with cache control headers
app.use(express.static(path.join(__dirname, 'build'), {
  maxAge: '1d', // Cache for 1 day
  setHeaders: (res, path) => {
    // Set cache-control headers based on file types
    if (path.endsWith('.js')) {
      res.setHeader('Cache-Control', 'public, max-age=31536000'); // 1 year
    } else if (path.endsWith('.css')) {
      res.setHeader('Cache-Control', 'public, max-age=31536000'); // 1 year
    } else if (path.endsWith('.html')) {
      res.setHeader('Cache-Control', 'public, max-age=0'); // No cache
    } else {
      res.setHeader('Cache-Control', 'public, max-age=86400'); // 1 day
    }
  }
}));

// Kamal health check route
app.get('/up', (req, res) => {
  res.send(`<!DOCTYPE html><html><body style="background-color: green"></body></html>`);
});

// Send all requests to index.html to handle routing in React Router
app.get('*', (req, res) => {
  res.sendFile(path.resolve(__dirname, 'build', 'index.html'));
});

// Start the server
app.listen(port, () => {
  console.log(`Server is running on port ${port}`);
});
```

We only need ExpressJS and the compression packages to run this small server. That's why we are including only these two packages in the runtime package file.

`package.runtime.json`

```json
{
  "name": "your-app",
  "version": "1.0.0",
  "main": "server.js",
  "dependencies": {
    "compression": "^1.7.4",
    "express": "^4.16.4"
  },
  "scripts": {
    "start": "node server.js"
  }
}
```



## GitHub Action

It's a headache to run the deployment from your own machine. The following GitHub Actions config will solve it for you. I have added a concurrency config too that will let only one deploy run at a time. It's useful when you make multiple commits in a short time.

```yaml
# .github/workflows/deploy.yml

name: Deploy to production

# To make sure that only one deploy runs at a time. Deploy lock will not let simultaneous deployments.
concurrency:
  group: ${{ github.workflow }}

on:
  push:
    branches: [master]

jobs:
  deploy:
    runs-on: ubuntu-latest
    env:
      KAMAL_REGISTRY_PASSWORD: ${{ secrets.KAMAL_REGISTRY_PASSWORD }}

    steps:
      - name: Set up Docker Buildx for cache
        uses: docker/setup-buildx-action@v3

      # Since we are using GHA cache, we need to expose the cache to the runtime
      - name: Expose GitHub Runtime for cache
        uses: crazy-max/ghaction-github-runtime@v3

      - name: Checkout code
        uses: actions/checkout@v3

      # Ruby is only needed to install Kamal
      - name: Set up Ruby
        uses: ruby/setup-ruby@v1
        with:
          ruby-version: '3.3'

      - name: Install dependencies
        run: gem install kamal

      # This is to facilitate Kamal with the private key to access server(s)
      - uses: webfactory/ssh-agent@v0.7.0
        with:
          ssh-private-key: ${{ secrets.SSH_PRIVATE_KEY }}

      - name: Run deploy command
        run: kamal deploy
```
