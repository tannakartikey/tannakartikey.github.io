---
layout: post
title: How to deploy multi-environment(staging, production) application using Kamal(previously MRSK)
image: "/assets/how-to-deploy-multi-environment-staging-production-application-using-mrsk.png"
tags: mrsk kamal rails postgresql
date: 2023-04-09 14:15 +0530
---
In the [previous post](https://www.kartikey.dev/2023/04/05/how-to-deploy-rails-app-and-postgres-with-mrsk-on-single-server.html){:target="_blank"}, I described how to host a Rails app and a database on a single server. This post will describe what if you want to host multiple environments i.e. staging, production of the same application using [Kamal](https://kamal-deploy.org/){:target="_blank"}.

Kamal uses [Traefik](https://traefik.io/){:target="_blank"} as a reverse proxy. It means that any incoming request will be handled by Traefik. Traefik will handover the request to appropriate server and Docker container based on the configuration. For example, let's assume that we have deployed the staging and the production version of our apps on the server. Then we have to configure Traefik in such a way that it points "staging.myapp.com" and "production.myapp.com" to the staging and the production Docker containers respectively.

Kamal uses Traefik with "docker" as a [provider](https://doc.traefik.io/traefik/providers/overview/){:target="_blank"}. Traefik [rules](https://doc.traefik.io/traefik/routing/routers/#rule){:taget="_blank"} are configured by providing certain lables to Docker containers. You can read more about it in their [docs](https://doc.traefik.io/traefik/providers/docker/){:target="_blank"}.

Let's start configuring Kamal. Kamal has a feature called ["destination"](https://github.com/mrsked/mrsk/pull/71){:target="_blank"}, we will use that to create two separate destinations - "staging" and "production". First, we will create the `config/deploy.yml` file with common options like service name, image name, registry, common environment variables, and Postgresql database as acessory.

```yml
# config/deploy.yml
service: myapp
image: user/myapp

registry:
  username: user
  password:
    - KAMAL_REGISTRY_PASSWORD

env:
  clear:
    DB_HOST: 99.99.99.99
  secret:
    - RAILS_MASTER_KEY
    - POSTGRES_PASSWORD

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

#### Staging config
Now, let's create a destination named "staging" with the `config/deploy.staging.yml` file.
```yml
# config/deploy.staging.yml
servers:
  - 99.99.99.99

labels:
  traefik.http.routers.myapp-web-staging.rule: Host(`staging.myapp.com`)

env:
  clear:
    POSTGRES_DB: myapp_staging
    IS_STAGING: true
```

In this file we have declared the server for the staging environment, environment variable spcific to the environment and we have used the `labels` key that will attach the labels to the staging Docker container.

Let's understand these rules with little more details. Kamal applies some default Traefik labels to each container in the "service-role-destination" format. With the help of these labels Traefik defines a [service](https://doc.traefik.io/traefik/routing/services/){:target="_blank"} (not to be confused with Kamal service). With the rule mentioned in the `deploy.staging.yml` file above, we are overriding the default labels. Since we have not specified any role, Kamal will assign the web role to the service. Anyways, there has to have one "web" role if we are specifying roles.

Don't forget to replace "staging.myapp.com" with your domain. The domain should be configured to point to the IP of the server.

#### Production config
```yml
# config/deploy.production.yml
servers:
  - 99.99.99.99

labels:
  traefik.http.routers.myapp-web-production.rule: Host(`rails.myapp.com`)

env:
  clear:
    POSTGRES_DB: myapp_production
```
The `deploy.production.yml` is similar to the staging configuration. Since we are going to deploy both the staging and the production as well as Postgresql(the database) on the same server, the same IP is configured under the `hosts` key. Again, don't forget to configure your domain.


That's it! You should have both the staging and the production as well as the database deployed on the same or different server(s) and accessible from the domains you have set for each environment!
