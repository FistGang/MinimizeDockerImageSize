# MimimizeDockerImageSize

Just how can we do to miminize docker images (or container) size

## Let's go through each strategy with examples

1. **Use Minimal Base Images**: Instead of using a full-fledged Linux distribution like Ubuntu, we can use Alpine Linux, which is much smaller in size. Here's an example Dockerfile using Alpine Linux:

    ```Dockerfile
    FROM alpine:latest
    RUN apk --no-cache add nginx
    CMD ["nginx", "-g", "daemon off;"]
    ```

2. **Multi-Stage Builds**: Suppose we're building a Node.js application. We can use multi-stage builds to compile our application and then copy only the necessary artifacts to the final image. Here's an example Dockerfile:

    ```Dockerfile
    # Build Stage
    FROM node:14 AS build
    WORKDIR /app
    COPY package*.json ./
    RUN npm install
    COPY . .
    RUN npm run build

    # Production Stage
    FROM nginx:alpine
    COPY --from=build /app/dist /usr/share/nginx/html
    ```

3. **Optimize Dockerfile Instructions**: Combine multiple `RUN` commands into a single one to reduce the number of layers. For example:

    ```Dockerfile
    FROM alpine:latest
    RUN apk --no-cache update && \
        apk --no-cache add nginx
    ```

4. **Minimize Dependencies**: Only install necessary dependencies in our container. For example, if our application only needs Python and Flask, our Dockerfile might look like this:

    ```Dockerfile
    FROM python:3.9-alpine
    RUN pip install Flask
    COPY . /app
    WORKDIR /app
    CMD ["python", "app.py"]
    ```

5. **Cache Dependencies**: Place frequently changing instructions towards the bottom of the Dockerfile to leverage caching. For example, copy the application code as late as possible:

    ```Dockerfile
    FROM node:14 AS build
    WORKDIR /app
    COPY package*.json ./
    RUN npm install
    COPY . .
    ```

6. **Compress Artifacts**: Before copying files into the container, compress them to reduce size. For example, if we have large static files:

    ```Dockerfile
    FROM nginx:alpine
    COPY --from=builder /app/dist /usr/share/nginx/html
    ```

7. **Use `.dockerignore`**: Create a .dockerignore file to exclude unnecessary files and directories. For example, exclude `node_modules` directory for a Node.js application:

    ```
    node_modules
    .git
    ```

8. **Remove Unused Layers**: After building our image, remove any unnecessary layers. You can use tools like Docker Squash to squash layers together:

    ```bash
    docker-squash -t new-image old-image
    ```

9. **Optimize Runtime Configuration**: Configure our application to use resources efficiently at runtime. For example, set appropriate memory limits in our Docker Compose file:

    ```yaml
    services:
      web:
        image: nginx:alpine
        deploy:
          resources:
            limits:
              memory: 512M
    ```

10. **Regularly Update Base Images**: Ensure we regularly update our base images and dependencies to incorporate security patches. We can use tools like Docker Security Scanning to identify vulnerabilities in our images.
