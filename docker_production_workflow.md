# Creating a Production-Grade Workflow with Docker

I want to develop an app with Docker and be able to push it to an outside hosting service, like AWS.

### Flow Architecture

![Imgur](https://i.imgur.com/f1sMGEV.png)

- Download and install Node from official Node website.
- Once it's installed, run `node -v` to get the version.
- I created a new folder caller *workspace_proj*. Inside that folder, run `npx create-react-app frontend`

- I received multiple npm errors
- I installed the latest version of npm by running `npm install -g npm@9.8.1`

Running `npx create-react-app frontend` again gave the following output:

![Imgur](https://i.imgur.com/aREuh77.png)

![Imgur](https://i.imgur.com/84L93LC.png)

This created a directory called *frontend*. 

`cd` into the directory.

![Imgur](https://i.imgur.com/TdWrBC8.png)

- Run `npm run test`
- Run `npm run build`
- After the build command, directories have been added to pwd
- These are the app's dependencies

![Imgur](https://i.imgur.com/kFn2nVj.png)

- Running `npm run start` gives the below output and opens a new browser tab:

![Imgur](https://i.imgur.com/NHhwrZA.png)

![Imgur](https://i.imgur.com/4PljerC.png)

## Setting up the Dockerfile

There will be two separate Dockerfiles - one for app development and the other for production.

In `frontend/` create **Dockerfile.dev**.

```
FROM node:16-alpine

WORKDIR 'app/'

COPY package.json .

RUN npm install

COPY . .

CMD ["npm", "run", "start"]
```

The `docker build` command needs to be amended as there is no file called Dockerfile, only Dockerfile.dev.

I instead need to run:

```
docker build -f Dockerfile.dev
```
to specify the file I want to build.

It takes FOREVER to build because of the additional dependencies. Dependencies were automatically installed to our pwd, but I usually get my dependencies from the Docker image.

Since I essentially have two copies of dependencies, I should delete one. 

In `frontend/` I deleted the folder *node_modules*, and then rebuild.

Then,
```
docker run <image id>
``` 

I get this output:

![Imgur](https://i.imgur.com/NVYXGWt.png)

But when I go to localhost:3000 the site cannot be reached.

I need to expose the port in the run command and map it.

I need to run:
```
docker run -p 3000:3000 <image id>
```

Now it works on port 3000.

## Docker Volumes

The source code for the React App homepage is located in frontend -> src -> App.js.

If I edit that file to add a picture of Kipo, it doesn't appear on the app.

This is because the dependencies have been copied from my local folder to the temporary Docker container.

![Imgur](https://i.imgur.com/71ls8FJ.png)

There needs to be a way where I can update the source code and the app updates in realtime without having to rebuild the image or restarting the container.

The volume is going to set up a reference that's going to point back to my local machine to access the folders in my local machine.


I'm essentially setting up mapping from a folder in the temp container to a folder in my local computer.

![Imgur](https://i.imgur.com/ElFa4sj.png)

In my case, I ran:
```
docker run -p 3000:3000 -v /app/node_modules -v $(pwd):/app f70403cecccba5d118c8ceef2a40c7462fe0e10aa741422646c021fda37f743b
```

This started up the container and it's now running on localhost:3000 with my Kipo image.

Now, if I go to the source code and amend it, the page updates in real time!

## Doing the above with Docker Compose

Instead of using that ridiculously long command, I can create a Docker Compose file with the post settings and the volumes we need to encode inside the container.

In the pwd, I've created a compose.yaml file. 

```
version: '3'
services:
  react-app:
    build:
      context: .
      dockerfile: Dockerfile.dev
    ports:
      - "3000:3000"
    volumes:
      - /app/node_modules
      - .:/app
```

I need to specify the context since the Dockerfile isn't called Dockerfile. 

I can run `docker-compose up` to run the container.

## Troubleshooting
This all works, but the app doesn't update in real time when I edit the source code.

Steps attemped:
- Amending the compose.yaml fole to edit the volume mapping. Under `volumes:`, changed `- .:/app` to `- ./src:/app`. App didn't work at all since build files are in frontend/
- In compose.yaml, cchanged the name of app from `react-app` to `web` - still doesn't update in real time
- To stop and kill the containers I ran `docker-compose down -v --rmi "all"` and ran `docker-compose up` again - same as above
- I ran `docker-compose down -v --rmi "all"` and amended `- .:/app` to both `- .:/app:cached` and `- .:/app:delegated` (two separate tests) - same as above
- I ran `docker-compose down -v --rmi "all"` then `docker-compose build --no-cache`, then `docker-compose up` - same as above

### (ignore since I'm not currently using Ubuntu)Troubleshooting for Windows 
The frontend directory needs to be copied from my local machine to Ubuntu. I have done this by:
1. Opening Ubuntu
2. Running `explorer.exe .` - **remember the dot at the end!**
3. The above step opens up the home directory of Ubuntu in Windows Explorer. Drag and drop frontend/ directly from local folder to Ubuntu