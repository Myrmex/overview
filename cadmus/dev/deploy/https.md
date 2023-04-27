---
layout: page
title: Configuring HTTPS
subtitle: Cadmus Deployment
---

## Configuring HTTPS - Method 1

- no third party script required.

To use HTTPS, you need to make some changes to the previous configuration to use a [certificate](https://en.wikipedia.org/wiki/Public_key_certificate) and change the URIs protocol from HTTP to HTTPS. Of course, there are several other methods, but the one illustrated here just continues the above configuration.

### Getting a Certificate

First you need a **certificate**. Get it from somewhere (e.g. <https://letsencrypt.org>), and prepare it in these formats:

- CER with KEY.
- PFX: you can convert a CER (or other format) certificate with its key file into a PFX file using the Linux `openssl` tool, like this:

```bash
openssl pkcs12 -export -out certificate.pfx -inkey <NAME>.key -in <NAME>.cer
```

Here your mileage may vary according to the format you get for the certificate. See the [openssl manual](https://www.openssl.org/docs/manmaster/man1/) for more, e.g. to convert PEM into CRT and KEY:

```bash
openssl x509 -outform der -in fullchain.pem -out docker_it.crt
openssl rsa -outform der -in privkey.pem -out docker_it.key
```

Then, place the 3 files under some folder in your host, e.g.:

- my CER and KEY files are under `opt/cadmus/cert`.
- my PFX file is under `opt/cadmus/nginx`.

This is required because we are going to share these certificates with the services running inside the Docker containers via a Docker volume. Using a volume is the suggested way of making a certificate available to what's inside a Docker image, without having to build an image with the certificate inside it, which would not be secure, nor practical.

### Passing Certificates to the API

- reference: [ASP.NET CORE SSL in Docker](https://docs.microsoft.com/en-us/aspnet/core/security/docker-compose-https).

The next step is passing our certificates to the API service inside the Cadmus Docker stack.

>As an ASP.NET 6 API, the Cadmus API service uses [Kestrel](https://learn.microsoft.com/en-us/aspnet/core/fundamentals/servers/kestrel).

(1) in the API service of the Docker compose script add a **volume** to share the PFX certificate file:

```yml
volumes:
  - /opt/cadmus/cert/certificate.pfx:/app/Infrastructure/Certificate/certificate.pfx
```

This instruction means that the file `certificate.pfx` under folder `/opt/cadmus/cert` in our host machine will be shared via a Docker volume pointing to file `/app/Infrastructure/Certificate/certificate.pfx` in the Docker image file system. This location is where Kestrel (the web server used by Cadmus API) will be instructed to find the certificate.

(2) continuing in the Docker compose script, add these environment variables in the same API service section of the Docker script, to tell Kestrel about the certificate:

```yml
environment:
  - ASPNETCORE_URLS=https://+:443;http://+:80
  - ASPNETCORE_Kestrel__Certificates__Default__Path=/app/Infrastructure/Certificate/certificate.pfx
  - ASPNETCORE_Kestrel__Certificates__Default__Password=YOURCERTPASSWORD
```

Of course, replace `YOURCERTPASSWORD` with the password used for your certificate.

These variables tell Kestrel that the HTTPS port is 443, the HTTP port is 80, the certificate is found at the specified path, and its password is that specified.

>💥 Mind the number of underscore (`_`) characters in these variable names! Please note that `ASPNETCORE_` ends with 1 underscore, whereas all the other underscores in the variable name come with 2 of them.

(3) still in this `environment` section, be sure to add an HTTPS (rather than HTTP) allowed origin for CORS, e.g.:

```yml
  - ALLOWEDORIGINS__0=https://docker.somewhere.it
```

>The suffix `__0` here is just the index in the array of allowed origins. So here it happens to be `0`, because it's the first origin I put in my list. Should you have more entries, change this index accordingly, remembering that it's 0-based (so the first origin you add is `ALLOWEDORIGINS__0`, the second `ALLOWEDORIGINS__1`, etc.).

This is enough for the API service: all what we did was sharing our PFX certificate file with Kestrel running in the Docker container, and telling it where to find this certificate.

Also, given that our frontend web app will be served in HTTPS, we ensured that its URI is among the allowed origins for this API.

### Passing Certificates to the Frontend

The frontend app uses [NGINX](https://www.nginx.com/) to serve the web app. You can find the default NGINX configuration for it among the source files of the Cadmus web app, in its GitHub repository.

(1) rather than modifying this configuration and rebuild the image, we're going to replace it with a new one, using our CER and KEY certificates. In the new NGINX configuration, the `server` section is replaced with this one:

```nginx
server {
  listen 443 ssl;
  listen 80;
  server_name localhost;
  ssl_certificate docker_it.cer;
  ssl_certificate_key docker_it.key;

  gzip on;
  gzip_min_length 1000;
  gzip_proxied expired no-cache no-store private auth;
  gzip_types text/plain text/css application/json application/javascript application/x-javascript text/xml application/xml application/xml+rss text/javascript;

  location / {
    root /usr/share/nginx/html;
    index index.html index.htm;
    try_files $uri $uri/ /index.html;
  }

  error_page 500 502 503 504 /50x.html;
  location = /50x.html {
    root /usr/share/nginx/html;
  }
}
```

The relevant changes are:

- `listen 443 ssl` for HTTPS.
- the `ssl_certificate` lines to define the CER and KEY certificates to be used.

The rest of the section is unchanged. Save this new `nginx.conf` configuration file somewhere in your host, e.g. in `/opt/cadmus/nginx`.

(2) in the Docker compose script, head to the `cadmus-app` service section, and make these changes:

- ensure you have changed the image name to reference your production version for the app. The default image name in the GitHub source refers to the development version, which targets API in localhost. Your production version should change the `env.js` file so that API are located at their HTTPS URIs.
- ensure that both ports 80 and 443 are exposed.
- add volumes to pass the NGINX configuration and the CER and KEY files to the Docker container.

Here is a sample:

```yml
cadmus-app:
  restart: always
  image: vedph2020/cadmus-app:0.0.10-prod
  ports:
    - 80:80
    - 443:443
  depends_on:
    - cadmus-api
    - cadmus-db
  networks:
    - cadmus-network
  volumes:
    # https://www.techrepublic.com/article/how-to-enable-ssl-on-nginx/
    # overwrite nginx.conf to use SSL with certificates from the host
    - /opt/cadmus/nginx/nginx.conf:/etc/nginx/nginx.conf
    - /opt/cadmus/nginx/docker_it.cer:/etc/nginx/docker_it.cer
    - /opt/cadmus/nginx/docker_it.key:/etc/nginx/docker_it.key
```

## Configuring HTTPS - Method 2

- third party script: [acme companion](https://github.com/nginx-proxy/acme-companion)

Among others, an easier configuration option is using the [acme-companion](https://github.com/nginx-proxy/acme-companion), which relies on Docker API to automate most of the certificate procedures: essentially, once it is in place, you just have to start the Docker compose script to have this script automatically request and apply a certificate from [LetsEncrypt](https://letsencrypt.org), and renewing it when required.

### Setup DNS

The first step is getting a domain for your site, so that LetsEncrypt can see it. Let's assume that the domain name is `mydomain.org`, and that you want to setup the API endpoint at `api.mydomain.org`, and the application at `app.mydomain.org`. You should setup your hosting environment accordingly (e.g. adding `A` -alias- records to the DNS configuration, pointing to the server's IP address via `RDATA`), e.g.:

- `A` = `api.mydomain.org`, `RDATA`=`99.100.101.102`.
- `A` = `app.mydomain.org`, `RDATA`=`99.100.101.102`.

### Setup Acme Companion

>This summarizes the tutorial at <https://blog.ssdnodes.com/blog/host-multiple-ssl-websites-docker-nginx>, with changes I introduced to use the more recent dockerized version of the ACME companion, which further simplifies the process.

(1) create and enter the directory where to place a Docker compose script:

```bash
mkdir nginx-proxy
cd nginx-proxy
```

(2) create a network to be shared among containers:

```bash
docker network create nginx-proxy
```

(3) in the folder created at (1), create a `docker-compose.yml` file for NGINX. This will be used as the reverse proxy which redirects traffic in your host machine.

```yml
version: '2'

# version 2 required by volumes_from
# https://github.com/nginx-proxy/acme-companion/blob/main/docs/Docker-Compose.md

services:
  nginx-proxy:
    image: nginxproxy/nginx-proxy
    container_name: nginx-proxy
    ports:
      - "80:80"
      - "443:443"
    restart: always
    volumes:
      - conf:/etc/nginx/conf.d
      - vhost:/etc/nginx/vhost.d
      - html:/usr/share/nginx/html
      - certs:/etc/nginx/certs:ro
      - /var/run/docker.sock:/tmp/docker.sock:ro

  acme-companion:
    image: nginxproxy/acme-companion
    container_name: nginx-proxy-acme
    volumes_from:
      - nginx-proxy
    volumes:
      - certs:/etc/nginx/certs:rw
      - /var/run/docker.sock:/var/run/docker.sock:ro
      - acme:/etc/acme.sh
    environment:
      - DEFAULT_EMAIL=daniele.fusi@unive.it

volumes:
  conf:
  vhost:
  html:
  certs:
  acme:

# Do not forget to 'docker network create nginx-proxy' before launch, and to add '--network nginx-proxy' to proxied containers.

networks:
  default:
    name: nginx-proxy
    external: true
```

(4) run the script:

```bash
docker compose up -d
```

To confirm, run `docker ps`. You should see 2 running containers with these names:

- `nginx-proxy`
- `nginx-proxy-acme`

This procedure is done once. When in place, you can use this infrastructure to add as many docker-based services as you want, each with its own subdomain and certificate.

### Modify Docker Compose Script

To set up any containerized app to work with this proxy, you must configure its compose script as follows:

- add 3 environment variables: `VIRTUAL_HOST`, `LETSENCRYPT_HOST`, `LETSENCRYPT_EMAIL`.
- replace the Docker network (with `nginx-proxy`, cf. step 2 in the previous section).
- exposing port 80/443 (this is already done in the default script).

With reference to the default docker compose script, you should make these changes:

(1) in the `cadmus-api` section, add the above environment variables (under `environment`), e.g. (of course change their values to fit your own setup):

```yml
cadmus-api:
  # ... omitted for brevity
  environment:
    - VIRTUAL_HOST=api.mydomain.org
    - LETSENCRYPT_HOST=api.mydomain.org
    - LETSENCRYPT_EMAIL=myemail@mydomain.org
    - ALLOWEDORIGINS__0=https://app.mydomain.org
```

(2) in the `cadmus-app` section, do the same for the app subdomain:

```yml
cadmus-app:
  # ... omitted for brevity
  environment:
    - VIRTUAL_HOST=app.mydomain.org
    - LETSENCRYPT_HOST=app.mydomain.org
    - LETSENCRYPT_EMAIL=myemail@mydomain.org
```

(3) in all the sections, remove the original network of the script as we're going to replace it with `nginx-proxy`. This means deleting (or commenting out) these lines from `cadmus-db`, `cadmus-api`, `cadmus-app`; and deleting (or commenting out) the final `cadmus-network`

- remove from any service:

```yml
    networks:
      - cadmus-network
```

- remove at the end:

```yml
networks:
  cadmus-network:
    driver: bridge
```

(4) at the end, add the `nginx-proxy` network instead (note the added `external` value here):

```yml
networks:
  default:
    external: true
    name: nginx-proxy
```

(5) rebuild the Angular app (follow the instructions in its `README.md`), and then under `dist/env.js` (mind `dist` here!) change the URI of the API according to your new setup, and the `version` variable as you prefer, e.g.:

```js
(function (window) {
  window.__env = window.__env || {};
  window.__env.apiUrl = "https://api.mydomain.org/api/";
  window.__env.version = "0.0.11-prod";
})(this);
```

(6) build and push a Docker image for your production environment, e.g.:

```bash
docker build . -t vedph2020/cadmus-app:0.0.11-prod
docker push vedph2020/cadmus-app:0.0.11-prod
```

(7) change the name of the app image accordingly in the `cadmus-app` section:

```yml
cadmus-app:
  image: vedph2020/cadmus-app:0.0.11-prod
```

(8) start the docker compose script, and in some moments you will be able to navigate to your HTTPS domain for both API and app.

▶️ next: [backup](backup.md)
