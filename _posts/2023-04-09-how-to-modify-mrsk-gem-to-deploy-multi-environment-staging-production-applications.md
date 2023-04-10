---
layout: post
title: How to modify MRSK gem to deploy multi-environment(staging, production) applications
tags: mrsk rails postgresql
date: 2023-04-09 14:15 +0530
---
In the previous post, I described how to host a Rails app and a database on a single server. This post will describe what if you want to host multiple environments i.e. staging, production of the same application using MRSK.

First of all, MRSK does not support this out-of-the-box at the moment. I have submitted PRs and will see where it goes. MRSK is still in beta phase and it's being developed very actively. But don't get discouraged. The changes required in the source code are extremely easy to understand and follow. I am going to describe with as much details as possible. It will be fun to poke around the source code of the newly developed gem. I am going to keep this post updated to reflect the status of the feature in the gem.

Now, let me start with explaining the issue and why it needs some tweaks in the code. If you have not read the previous post or have not tried MRSK at all, I highly enrourage you to do so.

MRSK uses Traefik as a reverse proxy. It means that any incoming request will be handled by Traefik. Traefik will handover the request to appropriate server and Docker container based on the configuration. For example, let's assume that we have deployed the staging and the production version of our apps on the server. Then we have to configure Traefik so that it points staging.example.com and production.example.com to their respective docker containers.

MRSK uses Traefik with docker set as provider. Traefik rules are configured by providing certain lables to Docker containers. You can read more about it in their docs.

Now, what MRSK does is that it assigns some default labels to the containers that defines the names of the Traefik service (not to be confused with MRSK service). It is possible to provide custom labels but the custom labels are merged with the default labels. As a result, MRSK defines more than one labels to Docker containers and Traefik throws the error.

Let's understand by an example. The name of your MRSK service is "myapp". MRSK labels all the containers of all the roles and destinations with the "traefik.http.routers.myapp.rule" rule. Now, to associate staging.example.com with myapp-staging container, if you assign the lable "traefik.http.routers.myapp-staging.rule" then MRSK will try to assign both the labels to the Docker container. And you can not have more than one service declared for the same container.

The PR I have created resolves this issue. It labels containers in the "service-role-destination" format. So, if you have "myapp" service, "web" role, and "staging"destination, then the label assigned to the containers will be "traefik.http.routers.myapp-web-staging.rule". This way you can target any specific role and destination to override the rules.

Don't worry if it all does not make sense at the moment. It will get crystal clear once we looks at the configuration.

The configuration below will create two destinations - staging and production. They are deployed with roles "staging" and "production" respectively. We can skip setting a role if we wish to, but I wanted to be as explicit as possible.

```yml
# config/deploy.yml
service: myapp
image: user/myapp

registry:
  username: user
  password:
    - MRSK_REGISTRY_PASSWORD

env:
  clear:
    DB_HOST: 99.99.99.99
  secret:
    - RAILS_MASTER_KEY
    - POSTGRES_PASSWORD

# Only requried if you want to enable Traefik dashboard
traefik:
  options:
    publish:
      - 8080:8080
  args:
    api.dashboard: true
    api.insecure: true

accessories:
  db:
    image: postgres:15
    host: 99.99.99.99
    port: 5432
    env:
      clear:
        POSTGRES_USER: 'myapp_db_user'
      secret:
        - POSTGRES_PASSWORD
    directories:
      - data:/var/lib/postgresql/data
```
The `deploy.yml` file is straightforward. First few options like `service`, `image`, `registry` and `env` are explained on the homepage of MRSK.

Traefik has a beautiful dashboard to shows visualized configuration.MRSK is not configured to show it by default. It would consume some resources for sure and not a good idea to keep it on if you are deploying multiple environments on a small single server. It would be good to keep that open until your configuration is working. You can see the dashboard by accessing the port `8080` of your server with the above configuration.

In the last, I have set up Postgresql database using MRSK's accessory feature. You can use the same host or choose a different server for Postgresql based on your requirement. Don't forget to change the value of the `DB_HOST` environment variable.

```yml
# config/deploy.staging.yml
servers:
  staging:
    traefik: true
    hosts:
      - 99.99.99.99

labels:
  traefik.http.routers.myapp-staging-staging.rule: Host(`staging.myapp.com`)

env:
  clear:
    POSTGRES_DB: myapp_staging
    IS_STAGING: true
```
The `deploy.staging.yml` file has settings only related to the staging environment. With the `labels` key, we are configuring Traefik to create a new service named "myapp-staging-staging"(in service-role-destination format) and redirect requests coming staging.myapp.com to myapp-staging-staging container. MRSK will merge the `env` values of the staging with the `env` values of the `config/deploy.md` files during deployment. Don't forget to replace "staging.myapp.com" with your actual domain. The domain's A record should point to the IP address you have set under the `hosts` key.

```yml
# config/deploy.production.yml
servers:
  production:
    traefik: true
    hosts:
      - 99.99.99.99

labels:
  traefik.http.routers.myapp-production-production.rule: Host(`rails.myapp.com`)

env:
  clear:
    POSTGRES_DB: myapp_production
```
The `deploy.production.yml` is similar to the staging configuration. Since we are going to deploy both the staging and the production as well as Postgresql(the database) on the same server, the same IP is configured under the `hosts` key. Again, don't forget to configure your domain.

After the configuration files are created, we need to use the local version of the MRSK gem. First of all, clone the mrsked/mrsk repository, preferably outside the project folder for convenience. Then we need to checkout the pull request I have created that supports environment-specific Traefik rules. If you use GitHub CLI then you can simply fire `gh pr checkout 197` from the MRSK clone. If you do not use GitHub CLI then you can do it with the following command:
```sh
git fetch upstream pull/197/head:traefik_rules_with_destination
```

After the branch is checked out, fire `bundle install` to install the gems required by MRSK gem.

You are now ready to use the local version of the gem. Swtich back to the project directory. Let's assume that MRSK is in the "~/source/mrsk" directory on your filesystem. You can use the local version of the MRSK gem in the following way:
```sh
# replace ~/source/mrsk with your own installation path
ruby -I "~/source/mrsk/lib" ~/source/mrsk/bin/mrsk version
```

If the above command succeeds then it means that you are successfully using the local copy of the modified gem. There are alternative ways to do this too, like mentioning the `path` for MRSK in Gemfile or building and installing the gem locally. But we are not doing that because I found this to be the easiest way that does not affect the implementation.

Now, the moment of truth! Let's start deploying our application. If you are deploying for the first time, then we need to use the `setup` command as described below:
```sh
ruby -I "~/source/mrsk/lib" ~/source/mrsk/bin/mrsk setup -d staging
```

Once the setup is completed, we need to deploy the production as well with:
```sh
ruby -I "~/source/mrsk/lib" ~/source/mrsk/bin/mrsk setup -d production
```

That's it! You should have both the staging and the production as well as the database deployed on the server and accessible from the domains you have set for each environment!
