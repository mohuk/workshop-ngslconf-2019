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
