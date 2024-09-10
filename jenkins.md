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

## Basic Jenkins Job

I am building a freestyle project called *Hello - freestyle*.

Go to *Build Steps* and select *Execute Shell*. Then run an `echo` command.

![Imgur](https://i.imgur.com/nsKF7g3.png)