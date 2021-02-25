# yadi

**Y**et **A**nother **D**ocker **I**mage repo

Collection of fine-tuned, production ready, multi-stage docker images for the major frameworks

All images are based on their `alpine` versions for lightweight and fast containers.

## Web

- [Next.js](./web/next.js/)
- [Nest.js](./web/nest.js/)
- [Create-React-App](./web/create-react-app/)

## Source images

- https://hub.docker.com/_/node

## Build multi-stage build explained

### Stage `base`

```Dockerfile
# Build target: base #
######################

FROM node:14-alpine AS base

# Default directory for all stages
WORKDIR /app

# We set the only build arg as default ENV
ARG NODE_ENV=production
ENV PATH=/app/node_modules/.bin:$PATH \
    NODE_ENV="$NODE_ENV"

# Copy only the package and lock files to be ready for installation
COPY package.json package-lock.json yarn.lock /app/

# Expose default port for the app
EXPOSE 3000
```

### Stage `dependencies`

```Dockerfile
# Build target dependencies #
#############################

FROM base AS dependencies

# Install prod dependencies
RUN yarn install --production && \

    # Cache prod dependencies
    # Later on we'll copy only this folder in the final prod image
    cp -R node_modules /prod_node_modules && \

    # Install dev dependencies
    # We need those for the build stage
    # After the build stage they will be discarded as layer is not used anymore
    # This will install only the delta dependencies between prod and dev
    yarn install --production=false
```


### Stage `development`

```Dockerfile
# Build target development #
############################

FROM dependencies AS development

# We copy the app sources
# Only what's left after .dockerignore is applied
# We skip node_modules/ as in dev env we normally mount them externally
COPY . /app

# Start the dev server
CMD [ "yarn", "dev" ]
```

### Stage `builder`

```Dockerfile
# Build target builder #
########################

FROM base AS builder

# We copy the dev dependencies from step `dependencies`
COPY --from=dependencies /app/node_modules /app/node_modules

# We copy the sources
COPY . /app

# Run lint so we fail early on errors
RUN yarn lint && \

    # When everything is good we start build process
    yarn build && \
    
    # Then we clean up the dev dependencies
    rm -rf node_modules
```

### Stage `production`

```Dockerfile
# Build target production #
###########################

# We start with clean base image with no dependencies
FROM base AS production

# Copy pre-built sources `dist`, `build` or `.next` depending on the framework
# In this case it's `.next` folder
COPY --from=builder /app/.next /app/.next

# Copy production dependencies only
COPY --from=dependencies /prod_node_modules /app/node_modules

# Copy app sources so any additional files are available
# This can be skipped in some scenarios
COPY . /app

# Start production server
CMD [ "yarn", "start" ]
```
