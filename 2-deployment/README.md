# Dockerfile for deployment
We are all aware that deploying a built Angular application does not require NodeJS or anything else to work. The application executes on the browser hence it only requires a webserver which can serve it to the world.

We are also aware that to build the app, we do need NodeJS and other packages. This eventually turns into a 2-stage process. To make sure our Docker images carry no noise packages/environment, we will build the deployment image as a multi-stage Docker file. Here goes:

# Stage 1: Build the Angular app

## Step 1
Create a file and name it `Dockerfile.build`. Since filename is difference than simply `Dockerfile`, it will need a mention via a flag when executing the commands.

```bash
$ touch Dockerfile
```

## Step 2
Populate the `Dockerfile.build` with the following contents

```bash
FROM node:10.16.3-alpine AS builder

RUN mkdir -p /docker-app
WORKDIR /docker-app
COPY package.json package-lock.json ./
RUN npm install

COPY . ./

RUN ./node_modules/.bin/ng build

FROM nginx:1.16.1-alpine

COPY --from=builder /docker-app/dist/* /usr/share/nginx/html
```

Let's break the instructions one by one.

```bash
# Base image should contain NodeJS published with the tag 10.16.3
# Tag in NodeJS images means the version of NodeJS installed in the image
# alpine is a very light weight linux distro
# this will spawn intermediate containers only to build the app
# it is aliased as "builder"
FROM node:10.16.3-alpine AS builder

# Create a new directory where we will move our code
RUN mkdir -p /docker-app

# Set the created directory as the default directory for future commands
WORKDIR /docker-app

# Copy the package.json package-lock.json into the Workdir
COPY package.json package-lock.json ./

# Run npm install to install all dependencies
RUN npm install

# Copy rest of the sorce code folders into the Workdir
COPY . ./

# Issue command to create a build from ng-cli
RUN ./node_modules/.bin/ng build

# Base image of webserver containing nginx version 1.16.1
# alpine is a very light weight linux distro
FROM nginx:1.16.1-alpine

# Copy build from builder image (above) "docker-app/dist/*" to html folder to serve via nginx
COPY --from=builder /docker-app/dist/* /usr/share/nginx/html
```

## Step 3
Let's compare the size of the 2 images by executing `docker images`. Find the 2 entries i.e. `docker-app` and `docker-app-build` and compare its `SIZE` column. This huge difference in size enables us to quickly transfer (push/pull) the image and greatly use deployment time.

# Providing Build Arguments as Angular Environments
If we want to make use of `--configuration` flag and not create a separate Dockerfile for each environment, we can make do by defining build arguments when building the Dockerfile.

## Step 1
This is the `Dockerfile.build` we created above.

```bash
FROM node:10.16.3-alpine AS builder

# Addition start
ARG CONFIG=dev
# Addition end

RUN mkdir -p /docker-app
WORKDIR /docker-app
COPY package.json package-lock.json ./
RUN npm install

COPY . ./

RUN ./node_modules/.bin/ng build --configuration ${CONFIG}

FROM nginx:1.16.1-alpine

COPY --from=builder /docker-app/dist/* /usr/share/nginx/html
```
