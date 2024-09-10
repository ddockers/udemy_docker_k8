# CI/CD with Jenkins

## Initial Setup

I'm usung the instructions provided by **vdespa** on GitHub at his repository https://github.com/vdespa/install-jenkins-docker.

CLone the repository.

`
git clone https://github.com/vdespa/install-jenkins-docker.git
`

This saved the repo to my home drive. There is a Dockerfile contained within that allows me to build the Jenkins Docker image,

` docker build -t my-jenkins .`

Once the build is complete, run `docker compose up -d`