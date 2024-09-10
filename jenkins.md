# CI/CD with Jenkins

## Initial Setup

I'm usung the instructions provided by **vdespa** on GitHub at his repository https://github.com/vdespa/install-jenkins-docker.

CLone the repository.

`
git clone https://github.com/vdespa/install-jenkins-docker.git
`

This saved the repo to my home drive. `cd` into the folder and there is a Dockerfile contained within that allows me to build the Jenkins Docker image.

` docker build -t my-jenkins .`

Once the build is complete, run `docker compose up -d`.

Jenkins is now running at http://localhost:8080. Setup is complete with username, password and default plugins.

To stop the Jenkins container running, run `docker compose down`.

### NB. When running `docker compose up -d` or `docker compose down` needs to be run from inside the *install-jenkins-docker* folder.

## Removing Jenkins

Run the following comand to terminate Jenkins and to remove all volumes and images used:

`docker compose down --volumes --rmi all `

## Basic Jenkins Freestyle Job

I am building a freestyle project called *Hello - freestyle*.

Go to *Build Steps* and select *Execute Shell*. Then run an `echo` command.

![Imgur](https://i.imgur.com/nsKF7g3.png)

Save.

The project is now in the dashboard. 

![Imgur](https://i.imgur.com/iGVBkte.png)

For the project to run, click on the project and select *Build Now*.

![Imgur](https://i.imgur.com/CObfc0P.png)

The console output is displayed in the build history of the project.

![Imgur](https://i.imgur.com/drgCu1G.png)

## Basic Jenkins Pipeline Job

I am creating a new pipeline project called *Hello - pipeline*.

A generic Hello World pipeline script can be created that looks like this:

![Imgur](https://imgur.com/BmZCIKX.png)

Save and build now. Here is the console output:

![Imgur](https://imgur.com/ay0E9P3.png)

The pipeline is used to keep track of changes. Instead of ```echo 'Hello World' ``` I can chance it to:

```
sh 'echo "Hello World"'
sh 'whoami'
```

This creates a new build and I can see the difference between each build.