# Using both development and production environments

To manage Node.js applications in both development and production environments using Docker Compose and multi-stage Dockerfiles, the `target` property in `docker-compose.yml` is used to specify which stage of the `Dockerfile` to build.

## 1. Multi-stage `Dockerfile`:

First, a multi-stage `Dockerfile` is created with distinct stages for development and production.

```Dockerfile
# Dockerfile
# Base stage
FROM node:20-alpine AS base
WORKDIR /app
COPY package*.json ./

# Development stage
FROM base AS dev
RUN npm install
COPY . .
CMD ["npm", "run", "dev"]

# Production stage
FROM base AS prod
RUN npm ci --only=production
COPY . .
RUN npm run build # If you have a build step for production
CMD ["node", "dist/index.js"] # Adjust based on your build output
```

## 2. Docker Compose Configuration:

Next, the `docker-compose.yml` file is configured to use the `target` property within the `build` section of the service definition.

```yml
# docker-compose.yml
version: "3.8"

services:
  nodejs-app:
    build:
      context: .
      dockerfile: Dockerfile
      target: ${NODE_ENV:-dev} # Default to 'dev' if NODE_ENV is not set
    ports:
      - "3000:3000" # Example port mapping
    volumes:
      - .:/app # Mount for development, potentially remove or modify for production
    environment:
      NODE_ENV: ${NODE_ENV:-development}
```

## 3. Building for Different Environments:

- **Development:** To build and run the development environment, the `NODE_ENV` environment variable is set to `dev` before running `docker compose up`.

```bash
NODE_ENV=dev docker compose up --build
```

This command will build the `dev` stage of the `Dockerfile` and mount the local source code into the container, allowing for live reloads or hot module replacement.

- **Production:** To build and run the production environment, `NODE_ENV` is set to `prod`.

```bash
NODE_ENV=prod docker compose up --build
```

This command will build the `prod` stage, which typically includes optimizations like installing only production dependencies and running a build process for compiled assets. The volume mount might be removed or adjusted for production deployments to avoid accidental changes to the running container.

By leveraging the `target` property and multi-stage builds, a single `Dockerfile` and `docker-compose.yml` can effectively manage both development and production environments for Node.js applications.
