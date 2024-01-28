---
layout: post
title: Deploy Rails app and Postgres with Kamal(previously MRSK) on single DigitalOcean server
image: "/assets/deploy-with-mrsk.png"
tags: mrsk rails postgresql
date: 2023-04-05 18:38 +0530
---
For me, it's been always cumbersome to host a Rails side project or personal application. Sure, Heroku is straightforward but it's not as cheap as I would like it to be. And I don't like the uptime limitation their free plan has. I have been making my way through Dokku and Capistrano. Luckily, a new door opened when [Kamal][1] was launched by DHH recently.

In this post, I am not getting into how Kamal works and the benefits of using it. DHH has created this fantastic [introduction video][2] for that.

Kamal is capable to deploy applications on multiple servers as well as a single server. In this post, I will describe how we can deploy a Rails application and Postgresql on a single VPS.

First of all, install and init Kamal on your local machine with instructions on the [README][3].

The key to the single server setup is to use the same hosts for the `web` as well as `accessories`. Here is what the `deploy.yml` file looks like for that setup:
```yml
service: my-app
image: user/my-app

servers:
  web:
    - 134.209.111.91

registry:
  username: tannakartikey
  password:
    - KAMAL_REGISTRY_PASSWORD

env:
  clear:
    DB_HOST: 134.209.111.91
  secret:
    - RAILS_MASTER_KEY
    - POSTGRES_PASSWORD

accessories:
  db:
    image: postgres:15
    host: 134.209.111.91
    port: 5432
    env:
      clear:
        POSTGRES_USER: 'my_app'
        POSTGRES_DB: 'my_app_production'
      secret:
        - POSTGRES_PASSWORD
    directories:
      - data:/var/lib/postgresql/data
```

I am using the default [`Dockerfile`][4] that is generated with Rails 7.1. Create and place the Dockerfile in your root directory if it does not exist already. You also need to create a [`.env`][5] file in the root of the repository and also need to make minor changes in your [`config/database.yml`][6] file.

I have created this sample [repository][7] that has all the code described in this post. This [commit][8] holds all the Kamal setup-related changes.

Be sure NOT to include your `.env` file in Git or your `Dockerfile`. If you are using Docker Hub or any other registry, make sure to make the image private because it holds a copy of the application code.

Next, we need to set up the server since we are deploying to the server for the first time. We can do it with:
```sh
bin/kamal setup
```
The `setup` command will set up all the accessories and deploy the app on the server.

That's it! You should have a running Rails app on the server which is connected to Postgresql on the same server. For subsequent deploy you can use `kamal deploy` or `kamal redeploy` based on your needs.

I have been using the USD4 droplet on DigitalOcean. If you are using a similar configuration I would recommend creating a [swap][9] partition on the server. I am assuming that if you are doing this, you are not expecting loads of traffic on day one. But without a swap partition, some simple operations like migrating the database may also fail.

It can also be possible to host the same code multiple times for different environments i.e. staging, production etc. For that, the ["role"][10] and ["destination"][11] functionality of Kamal can be used. I might cover that in a separate post.

[1]: https://kamal-deploy.org/
[2]: https://www.youtube.com/watch?v=LL1cV2FXZ5I
[3]: https://github.com/basecamp/kamal#readme
[4]: https://github.com/tannakartikey/rails_71_mrsk_deploy/blob/2d138e10b2c87ec0bccc382ae06ce6e71c6f7187/Dockerfile
[5]: https://github.com/tannakartikey/rails_71_mrsk_deploy/commit/2d138e10b2c87ec0bccc382ae06ce6e71c6f7187#diff-e9cbb0224c4a3d23a6019ba557e0cd568c1ad5e1582ff1e335fb7d99b7a1055d
[6]: https://github.com/tannakartikey/rails_71_mrsk_deploy/commit/2d138e10b2c87ec0bccc382ae06ce6e71c6f7187#diff-5a674c769541a71f2471a45c0e9dde911b4455344e3131bddc5a363701ba6325
[7]: https://github.com/tannakartikey/rails_71_mrsk_deploy
[8]: https://github.com/tannakartikey/rails_71_mrsk_deploy/commit/2d138e10b2c87ec0bccc382ae06ce6e71c6f7187
[9]: https://www.digitalocean.com/community/tutorials/how-to-add-swap-space-on-ubuntu-20-04
[10]: https://github.com/mrsked/mrsk/pull/99
[11]: https://github.com/mrsked/mrsk/pull/71
