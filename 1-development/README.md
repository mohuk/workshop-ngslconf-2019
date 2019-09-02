# Dockerfile for development

## Step 1
Create a new Angular project via the [angular-cli](https://www.npmjs.com/package/@angular/cli).

```bash
$ ng new docker-app --defaults
```

Run and verify if the application is working fine.

```bash
$ npm run start
```

Browse to `http://localhost:4200`, you should see the default Angular app page.

## Step 2
Create a file and name it `Dockerfile`. You can name the file anything but Docker recognizes the name `Dockerfile` by default. All other names would require a mention via a flag. We will just keep it `Dockerfile`.

```bash
$ touch Dockerfile
```

## Step 3
Create a file named, `.dockerignore`. This file will contain the list of files and folders which will be ignored by Docker commands.

```bash
$ touch .dockerignore
```

Put the following contents in the file

```code
node_modules
```

## Step 4
Think of the Docker container as a completely different machine. Let's start putting commands in the `Dockerfile`.

```code
FROM node:10.16.3

RUN mkdir -p /docker-app
WORKDIR /docker-app
COPY package.json package-lock.json ./
RUN npm install

COPY . ./

EXPOSE 4200

ENTRYPOINT ["./node_modules/.bin/ng"]
CMD ["serve", "--host","0.0.0.0"]
```

Let's break the instructions one by one.

```code
# Base image should contain NodeJS published with the tag 10.16.3
# Tag in NodeJS images means the version of NodeJS installed in the image
FROM node:10.16.3

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

# Expose port 4200 to make it available to be bound on the host machine
EXPOSE 4200

# Define an Entrypoint from where the commands can be executed
ENTRYPOINT ["./node_modules/.bin/ng"]

# Define a command to execute from the Entrypoint
CMD ["serve", "--host","0.0.0.0"]
```

`EXPOSE`, `ENTRYPOINT` and `CMD` can be overridden when exeuting the image.

## Step 5
Build the image with the `Dockerfile` with the following command

```bash
$ docker build --tag {app_tag_name} {root_context}
```

In our case this will be

```bash
$ docker build --tag docker-app .
```

When successful, execute the `docker images` command to see your image named `docker-app` in the list.
