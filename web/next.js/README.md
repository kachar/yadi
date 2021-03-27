# Next.js Dockerfile

Production-ready setup for Next.js projects

- <https://nextjs.org/>
- <https://github.com/vercel/next.js>

Based on `node:14-alpine`

## Development flow

Run the stack

```shell
docker-compose up -d
```

Visit <http://app.yadi.localhost:3070>

## Environment

Three sample environment files are included in the sample app. Normally only development mode `.env` file should be included in git as others might contain sensitive data.

- `.env`
- `.env.staging`
- `.env.prod`

### Docker Compose

With the `COMPOSE_FILE` we load two linked configurations in development mode.

```env
COMPOSE_FILE=docker-compose.yml:docker-compose.dev.yml
```

Note: On Windows use `;` as file [separator](https://docs.docker.com/compose/reference/envvars/#compose_file) instead of `:`

#### Main `docker-compose.yml`

The main configuration defines:

- core services specs
- variables used in all environments
- networking

#### Development `docker-compose.dev.yml`

Dev configuration inherits and extends the main one and adds:

- image build instructions
- local mounted volumes
- exposed ports to host machine
- local container hotname (app.yadi.localhost)

### Application environment

Custom application environment, feel free to adjust or remove if not used.

```env
APP_ENV=development # development, staging, production
```

### Node environment

Node env enables special code optimizations in production builds. Application environment such as `staging` normally will benefit of the code optimizations.
Better [not to be used](https://glebbahmutov.com/blog/do-not-use-node-env-for-staging/) as application environment.

```env
NODE_ENV=development # development, production
```

### Multi-stage build step

Target environment defines the end stage of the [multi-stage build](https://docs.docker.com/develop/develop-images/multistage-build/)

You can read more about the different stages in the main [README.md](https://github.com/kachar/yadi)

```env
TARGET_ENV=development # dependencies, development, builder, production
```

### Combinations

| `APP_ENV`   | `NODE_ENV`  | `TARGET_ENV` | Notes                                                   |
| ----------- | ----------- | ------------ | ------------------------------------------------------- |
| development | development | development  | Normal development flow                                 |
| development | production  | production   | Local production build                                  |
| staging     | development | development  | Not very useful, can help reach certain code conditions |
| staging     | production  | production   | Production build deployed on staging server             |
| production  | production  | production   | We're live                                              |

### Alternatives

- <https://github.com/vercel/next.js/tree/canary/examples/with-docker>
