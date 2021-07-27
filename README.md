# traefik_container_reverse_proxy
Repository for my container reverse proxy setup with Træfik

## 1) generate admin password

```bash
htpasswd -nb admin YOUR_PASSWORD

```

## 2) traefik_dynamic.toml

- change **YOUR_ADMIN_PASSWORD** to the output from step 1
- change **SUBDOMAIN.YOUR-DOMAIN.COM** to your traefik url (e.g. traefik.example.com)

```bash
[http.middlewares.simpleAuth.basicAuth]
  users = [
    "admin:YOUR_ADMIN_PASSWORD"
  ]

[http.routers.api]
  rule = "Host(`SUBDOMAIN.YOUR-DOMAIN.COM`)"
  entrypoints = ["websecure"]
  middlewares = ["simpleAuth"]
  service = "api@internal"
  [http.routers.api.tls]
    certResolver = "lets-encrypt"

```

## 3) create acme file

for storing the information on the certificates, we need the acme file

```bash
touch acme.json
chmod 600 acme.json
```

## 4) traefik.toml

change your email address in traefik.toml:
```bash
...
[certificatesResolvers.lets-encrypt.acme]
  email = "your.name@email.com"
  storage = "acme.json"
  tlschallenge=true
...
```

# add applications

## modify docker-compose

besides the regular container setup, the labels for Træfik are important. Change the following fields accordingly:
- **YOUR_ROUTER** -> the name of your configuration router
- **APP-SUBDOMAIN.YOUR-DOMAIN.COM** -> the full url for your container, e.g. app1.example.com
- traefik.port -> if you expose a port, adjust it

```bash
...
labels:
      - traefik.http.routers.YOUR_ROUTER.rule=Host(`APP-SUBDOMAIN.YOUR-DOMAIN.COM`)
      - traefik.http.routers.YOUR_ROUTER.tls=true
      - traefik.http.routers.YOUR_ROUTER.tls.certresolver=lets-encrypt
      - traefik.http.routers.YOUR_ROUTER.tls.domains[0].main=APP-SUBDOMAIN.YOUR-DOMAIN.COM
      - traefik.port=8080
    ports:
      - "8080:8080" #example
...
```