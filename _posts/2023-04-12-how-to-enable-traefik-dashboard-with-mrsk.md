---
layout: post
title: How to enable Traefik dashboard with MRSK
image: assets/traefik-dashboard.png
tags: mrsk
date: 2023-04-12 18:40 +0530
---
[ MRSK ](https://mrsk.dev){:target="_blank"} uses Traefik as a dynamic reverse-proxy. Traefik has a beautiful [ dashboard ](https://doc.traefik.io/traefik/operations/dashboard/){:target="_blank"} to visually display the configuration. MRSK is not configured to show the dashboard by default. 

Traefik dashboard can run in two modes - secure and insecure. Traefik recommends secure mode but this post wil cover how to enable both the modes.


First, let's see the unsecure mode. Unsecure mode is very easy to configure. It is unsecure because it exposes Traefix API on the [ entrypoint ](https://doc.traefik.io/traefik/routing/entrypoints/){:target="_blank"}. It means that after configuring the dashboard in unsecure mode, the dashboard will be available on the port 8080 of the host.

Adding the following snippet in the `config/deploy.yml` file will enable the unsecure dashboard.

```yml
# config/deploy.yml
traefik:
  options:
    publish:
      - 8080:8080
  args:
    api.dashboard: true
    api.insecure: true
```
After adding the configuration, run the `mrsk traefik reboot` command to apply the configuration. This command will stop, remove and start new container again with the latest configuration. The dashboard should be visible on the port `8080` of the host server now e.g. http://99.99.99.99:8080


Now, let's see about the secure mode. It's called secure mode because the API is not expose on the entrypoint. We need to create a router rule that uses `api@internal` service. Let's look at the configuration for the secure model.

```yml
traefik:
  options:
    publish:
      - 8080:8080
  args:
    api.dashboard: true
  labels:
    traefik.enable: "true"
    traefik.http.routers.dashboard.rule: Host(`traefik.example.com`) && (PathPrefix(`/api`) || PathPrefix(`/dashboard`))
    traefik.http.routers.dashboard.service: "api@internal"
    traefik.http.routers.dashboard.middlewares: "auth"
    traefik.http.middlewares.auth.basicauth.users: test:$2y$05$H2o72tMaO.TwY1wNQUV1K.fhjRgLHRDWohFvUZOJHBEtUXNKrqUKi
```
Here, we have configured Traefik dynamically with help of Docker labels. First of we have created a router. Then we attached the router with the `api@internal` service because in the secure mode we have to do this manually. After that, we added the auth middleware to Trafeik. In the last, we conifgured this `auth` middleware to use [HTTP Basic Authentication](https://doc.traefik.io/traefik/middlewares/http/basicauth/){:target="_blank"} and provided it with the credentials. You can read more about the rules in the details on the Traefik [ docs ](https://doc.traefik.io/traefik/routing/routers/){:target="_blank"}.

The credentials are in the "username:hashed_password" format. The credentials are generated with the `htpasswd` command. Let's say you want to create a user with the username "admin" and the password "super_strong_password" then you can use the following command:
```sh
htpasswd -nb admin super_strong_password
# output: admin:$apr1$2FGO09Gu$PSZdmmJqyrXWYvidWAm6p0
```

You will get the password hash in the output. Just copy paste the output with the username:password in the labels. The official Traefik docs mention that you need to escape the `$` character but you don't need if you are using MRSK but MRSK [ escapes ](https://github.com/mrsked/mrsk#using-shell-expansion){:target="_blank"} the `$` sign for these labels.

That's it! Don't forget to reboot the Trafeik container with the `mrsk traefik reboot` comamnd. After that, the dashboard should be accessible on the http://traefik.example.com/dashboard endpoint.

P.S. I would not recommend running the dashboard permanently if you are using a small single server with multple apps. Because that would eat up the valuable server resources. 
