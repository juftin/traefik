# Traefik

<p align="left">
  <img src="docs/static/juftin.png" width="125" height="125"  alt="juftin logo">
  <img src="https://github.com/traefik/traefik/raw/master/docs/content/assets/img/traefik.logo.png" width="150" alt="traefik logo">
</p>

Hosted Reverse Proxy with Google OAuth for operating a webserver
from home. This reverse-proxy automatically picks up new docker services
given the proper labels are applied.

-   [Configuration](#configuration)
    -   [Port Forwarding](#port-forwarding)
    -   [CloudFlare](#cloudflare)
    -   [Google OAuth 2.0](#google-oauth-20)
    -   [DuckDNS](#duckdns)
    -   [File Configuration](#file-configuration)
        -   [.env](#env)
        -   [acme.json](#acmejson)
-   [Usage](#usage)
    - [Jupyter Example](#jupyter-example)
    - [Local Usage](#local-usage)

* * *

* * *

### ❗️ Important Notice ❗

This docker-compose application stack is meant to host a website online using 
cloudflare as a DNS provider. Separately, this repository has a [local](https://github.com/juftin/traefik/tree/local)
branch used for running a reverse Proxy locally (i.e. `http://jupyter.localhost/`)

## Configuration

### Port Forwarding

In order to reach the outside world, you must forward ports 
`80` and `443` from your server IP address through your router. 
See your router's manual for Instructions.

### CloudFlare

Instructions in this article to set up 
free [CloudFlare](https://dash.cloudflare.com/sign-up) services. 
The CloudFlare section of the article can be found 
[here](https://www.smarthomebeginner.com/traefik-reverse-proxy-tutorial-for-docker/#Dynamic_DNS_or_Your_Own_Domain_Name).

### Google OAuth 2.0

The Google Oauth 2.0 configuration can be found [here](https://www.smarthomebeginner.com/google-oauth-with-traefik-docker/#How_do_I_setup_OAuth). This configuration requires Google Authentication to access any services published on the web.

### DuckDNS

A free DuckDNS dynamic DNS subdomain can be set up [here](https://www.duckdns.org).

### File Configuration

#### `.env`

The [`media-center.yml`](media-center-juftin-personal.yml) file can be left alone if the default list of services 
is acceptable. The [`example.env`](example.env) file can be modified and renamed `.env` in order 
for the containers to be build properly. This is the entire configuration file for
all applications. All relevant hints can be found within.

#### `acme.json`

You will need to create an empty [acme.json](traefik/acme/acme.json) file for the
application to work and generate an SSL Certificate through LetsEncrypt. 
However, while initially setting up it will be useful to remove and recreate the file to force
certificate recreation. Keep in mind that certificate creation and registration can take some tie.
uncomment the `certificatesResolvers.dns-cloudflare.acme.caServer=https://acme-staging-v02.api.letsencrypt.org/directory` 
command on the traefik service in the [docker-compose](media-center-juftin-personal.yml) file while testing. 
The instructions are below:

  - file location: [`traefik/config/acme/acme.json`](traefik/acme/acme.json)
  - file permissions (chmod): `600`

```shell script
mkdir -p traefik/acme/ && \
  rm -f traefik/acme/acme.json && \
  touch traefik/acme/acme.json && \
  chmod 600 traefik/acme/acme.json
```

## Usage

### Jupyter Example:

The below example allows the jupyter container to speak to the `traefik_reverse-proxy` (the 
external network created by this *compose* configuration - notice the docker-compose 
project name added before the network name). Apart from the networking, everything 
else is performed with the labels.

```yaml
version: '3.7'
networks:
    traefik:
        external:
            name: traefik_reverse-proxy
services:
    jupyter:
        container_name: jupyter
        image: jupyter/all-spark-notebook:latest
        networks:
            traefik: null
        command:        start.sh jupyter lab
        labels:
            traefik.enable:                                             true
            traefik.http.routers.jupyter-rtr.rule:                      Host(`jupyter.${DOMAIN_NAME}`)
            traefik.http.routers.jupyter-rtr.service:                   jupyter-svc
            traefik.http.services.jupyter-svc.loadbalancer.server.port: 8888
            traefik.http.routers.jupyter-rtr.entrypoints:               https
            traefik.http.routers.jupyter-rtr.middlewares:               chain-oauth-google@file
```

- `traefik.enable`
    - Allows Traefik to interact with this application 
- `traefik.http.routers.jupyter-rtr.rule`
    - Creates a router, "jupyter-rtr", that can be accessed @ jupyter.example.com
- `traefik.http.routers.jupyter-rtr.service`
    - Attaches a load balancing service, "jupyter-svc",to the router
- `traefik.http.services.jupyter-svc.loadbalancer.server.port`
    - Instructs the load balancer to operate on port 8888 (the exposed port of the application)
- `traefik.http.routers.jupyter-rtr.entrypoints`
    - Instructs the router to use the "https" entrypoint (https://jupyter.example.com)
- `traefik.http.routers.jupyter-rtr.middlewares:`              
    - Instructs the router to use the middleware service, 
    "[chain-oauth-google@file](traefik/rules/middlewares-chains.yml)",
    which requires Google OAuth for access 

### Local Usage

Check out the [local](https://github.com/juftin/traefik/blob/local/docker-compose.yml)
branch of this repository for a configuration that supports using a local reverse proxy. 

Most importantly, for the above example, the `traefik.http.routers.jupyter-rtr.entrypoints` would 
be set to `http` (since we will not be using SSL locally), and the `traefik.http.routers.jupyter-rtr.middlewares`
value can be set to "[chain-local-testing@file](traefik/rules/middlewares-chains.yml)"
to bypass Google OAuth.

# Containers

-   [traefik](#traefik)
-   [oauth](#oauth)
-   [duckdns](#duckdns)

## traefik

[Docker Hub](https://hub.docker.com/_/traefik) \|\|
[GitHub](https://github.com/containous/traefik) \|\|
[Website](https://traefik.io/) \|\|
[Documentation](https://docs.traefik.io/)

<img src="docs/static/traefik_logo.png" width="300" alt="traefik Logo">

Traefik (pronounced traffic) is a modern HTTP reverse proxy 
and load balancer that makes deploying microservices easy. 
Traefik integrates with your existing infrastructure components 
(Docker, Swarm mode, Kubernetes, Marathon, Consul, Etcd, Rancher, Amazon ECS, ...) 
and configures itself automatically and dynamically. Pointing Traefik 
at your orchestrator should be the only configuration step you need.

## oauth

[Docker Hub](https://hub.docker.com/r/thomseddon/traefik-forward-auth) \|\|
[GitHub](https://github.com/thomseddon/traefik-forward-auth)

<img src="docs/static/oauth2.png" width="250" alt="oauth">

A minimal forward authentication service that provides Google oauth based 
login and authentication for the traefik reverse proxy/load balancer.

## duckdns

[Docker Hub](https://hub.docker.com/r/linuxserver/duckdns/) \|\|
[GitHub](https://github.com/linuxserver/docker-duckdns) \|\|
[Website](https://www.duckdns.org)

<img src="docs/static/duckdns.jpg" width="250" alt="duckdns">

Duckdns is a free service which will point a DNS (sub domains of duckdns.org) 
to an IP of your choice. The service is completely free, and doesn't 
require reactivation or forum posts to maintain its existence.

* * *

* * *

### Special Thank You

This configuration was inspired by, and 
immensely helped by the article at 
[https://smarthomebeginner.com](https://www.smarthomebeginner.com/traefik-2-docker-tutorial). 


[Here](https://github.com/htpcBeginner/docker-traefik) 
is their massive home server setup on GitHub, and the accompanying 
[docker-compose](https://github.com/htpcBeginner/docker-traefik/blob/master/docker-compose-t2.yml) 
file.

* * *

* * *

<br/>
<br/>
<br/>

###### Cool stuff happens in Denver, CO [<img src="https://upload.wikimedia.org/wikipedia/commons/thumb/6/61/Flag_of_Denver%2C_Colorado.svg/800px-Flag_of_Denver%2C_Colorado.svg.png" width="25" alt="Denver">](https://denver-devs.slack.com/)