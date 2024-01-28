---
layout: post
title: How to deploy a NodeJS application using Kamal (previously MRSK)
tags: mrsk
date: 2023-04-10 13:55 +0530
---
[Kamal](https://kamal-deploy.org/) is created by DHH - founder of Rails, but the tool is not limited to deploy only Rails applications. Kamal is a simple tool that automates some Docker related commands. That is why it can support any platform/language. In this post, let's see how to deploy a NodeJS application on a small DigitalOcean VPS using Kamal.

First of all, let's generate an express app. I installed [`express-generator`](https://expressjs.com/en/starter/generator.html){:target="_blank"} and generated a new app using the `express express-app` command.

Then let's create a Dockerfile:
```
FROM node:16

# Create app directory
WORKDIR /usr/src/app

# Install app dependencies
# A wildcard is used to ensure both package.json AND package-lock.json are copied
# where available (npm@5+)
COPY package*.json ./

RUN npm install
# If you are building your code for production
# RUN npm ci --omit=dev

# Bundle app source
COPY . .

EXPOSE 3000
CMD [ "node", "bin/www" ]
```

Now, let's use Kamal to generate the config files. Make sure you have Kamal installed on your machine. If not, you can [ follow these instructions ](https://kamal-deploy.org/docs/installation) on README.

Let's init the configuration with `kamal init`. It will create `config/deploy.yml` and `.env` files in your root directory. My config file looks like this:

```yml
service: express-app

image: user/express-app
servers:
  - 146.190.86.77

registry:
  username: user
  password:
    - KAMAL_REGISTRY_PASSWORD

healthcheck:
  path: /
  port: 3000
```

The config file I have created here is minimal. It defines a service, provides the server IP, Docker Hub registry info and healthcheck information. After successful deploy, Kamal will ping the "/" path on port "3000" and will expect the "200 OK" response from the server.

Now, let's deploy the app with the `kamal deploy` command. Kamal will deploy your app and you can check if it is live or not by entering the server IP address in the browser. If you want to connect your app with the database, or host multiple environment of the application like staging, production then you can check out my [ previous posts ](https://www.kartikey.dev/tag/mrsk/). They are platform agnostic and applicable on NodeJS apps as well.
