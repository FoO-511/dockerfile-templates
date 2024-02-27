# `next.js standalone build` with `yarn berry` using `private npm` and `npm scope`

`next.js standalone build` is quite incompatible pnp so I use node-modules instead.

## usage

You can use it like:

```bash
make build tag=qa env=test
```

You need to make `.env.test` file to make docker build work properly.(if you set `env=test` option.)

If you only use `.env` you can disable `COPY .env.$STAGE .env.local` in the `Dockerfile`

## info

### tested vertions

- `nextjs` : `14.1.0`
- `yarn` : `3.6.4`
- `docker` : `24.0.7`
- `make` : `3.81`

### yarn config set

```dockerfile
RUN yarn config set "npmRegistries['${NPM_REGISTRY_HTTP}'].npmAlwaysAuth" true \
&& yarn config set "npmRegistries['${NPM_REGISTRY_HTTP}'].npmAuthToken" ${NPM_TOKEN} \
&& yarn config set "npmScopes['${NPM_SCOPE}'].npmAlwaysAuth" true \
&& yarn config set "npmScopes['${NPM_SCOPE}'].npmRegistryServer" ${NPM_REGISTRY_HTTP} \
&& yarn config set nodeLinker node-modules
```

This part from `Dockerfile` will make your `.yarnrc.yml` in docker build look like below.

```yaml
(...other options in your host .yarnrc.yml)

nodeLinker: node-modules

npmRegistries:
  "YOUR_NPM_DOMAIN":
    npmAlwaysAuth: true
    npmAuthToken: YOUR_TOKEN

npmScopes:
  YOUR_SCOPE:
    npmAlwaysAuth: true
    npmRegistryServer: "YOUR_NPM_DOMAIN"
```
