# Build target dependencies #
#############################
FROM node:14-alpine AS base
WORKDIR /app
ARG NODE_ENV=production
ENV PATH=/app/node_modules/.bin:$PATH \
    NODE_ENV="$NODE_ENV"
COPY package.json yarn.lock /app/
EXPOSE 3000

# Build target dependencies #
#############################
FROM base AS dependencies
# Install prod dependencies
RUN yarn install --production && \
    # Cache prod dependencies
    cp -R node_modules /prod_node_modules && \
    # Install dev dependencies
    yarn install --production=false

# Build target development #
############################
FROM dependencies AS development
COPY . /app
CMD ["yarn", "start"]

# Build target builder #
########################
FROM base AS builder
COPY --from=dependencies /prod_node_modules /app/node_modules
COPY . /app
RUN yarn build && \
    rm -rf node_modules

# Build target production #
###########################
FROM base AS production
RUN yarn global add serve --silent
COPY --from=builder /app/build /app/build
COPY --from=dependencies /prod_node_modules /app/node_modules
COPY . /app
CMD ["/bin/sh", "-c", "serve -s /app/build --listen 3000 --no-clipboard"]
