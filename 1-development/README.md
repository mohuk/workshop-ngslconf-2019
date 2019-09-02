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

```bash
FROM node:10.16.3-alpine

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

```bash
# Base image should contain NodeJS published with the tag 10.16.3
# Tag in NodeJS images means the version of NodeJS installed in the image
# alpine is a very light weight linux distro
FROM node:10.16.3-alpine

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
# Example command
$ docker build --tag {app_tag_name} {root_context}
```

In our case this will be

```bash
$ docker build --tag docker-app .
```

When successful, execute the `docker images` command to see your image named `docker-app` in the list.

## Step 6
Let's run the image created in `Step 5` with the following command

```bash
# Example command
$ docker build --detach --publish {host_port}:{container_port} {docker_image_tag}
```

Which in our case will be:

```bash
$ docker run --detach --publish 4200:4200 docker-app
```

Browse [localhost:4200](http://localhost:4200) and find your application running inside a container. To see the running container, run `docker ps` and find your container listed.

## Step 7
Notice that changing the files on the editor does not reflect on the app like it does on running ng serve on your machine instead of a container. Explanation for that is, the container is running a copy of the files instead of the original. Rebuilding and running the container will reflect the content. To link the `src` folder from host machine to container, we will make use of a volume mount.

```bash
# Example command
$ docker run --detach --publish {host_port}:{container_port} --volume {host_folder_path}:{container_folder_path} {docker_image_tag}
```

Which in our case will be:

```bash
docker run --detach --publish 4200:4200 --volume {absolute_path_to_folder}/src/:/docker-app/src docker-app
```

Change the source, save and see the changes reflect.

## Why would we ever need this complication ?
Our demonstration is fairly simple and is only educating the most basic of cases. But in real life, there might be additional packages our application might require outside of `npm` say, `make`, `gcc`, `g++`, `python` or `ruby`.

Did you know [Sass](https://sass-lang.com/) was originally written in `ruby`? Before we had [node-sass](https://github.com/sass/node-sass), apps that used [Sass](https://sass-lang.com/) required `ruby` and its respective gems installed. There are still packages which you might require that are ourside of `npm` e.g. [angularfire2](https://github.com/angular/angularfire2) when used with cloud functions requires a few of those packages. You never noticed because you use `Ubuntu` or `Windows` which might have them already installed. Build servers are generally bare bones and this is where Docker would take care of building the environment for a successful build.

Building such environments in containers guarantee that the correct envirnment is baked into the image for the application to run successfully.
