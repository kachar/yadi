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

## Build multi-stages explained

#### Stage `base`

```Dockerfile
# Build target: base #
######################
FROM node:14-alpine AS base

WORKDIR /app

# We set the only build arg as default ENV
ARG NODE_ENV=production
ENV PATH=/app/node_modules/.bin:$PATH \
    NODE_ENV="$NODE_ENV"

# Copy only the package and lock files to be ready for installation
COPY package.json package-lock.json yarn.lock /app/

# Expose default port for the container
EXPOSE 3000
```
