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

## Laptop Assembly Pipeline

Create a new job called *laptop assembly*.

Navigate to *Pipeline* and select the Hello World sample to give us the structure we need.

```
pipeline {
    agent any

    stages {
        stage('Hello') {
            steps {
                echo 'Hello World'
            }
        }
    }
}
```

Change `Hello` stage to `Build` stage. When building the laptop, we want to store everything in a directory.Then, we need to make a file called `computer.txt` that will be stored in the `build` directory.

```
pipeline {
    agent any

    stages {
        stage('Build') {
            steps {
                echo 'Building a new laptop...'
                sh 'mkdir build'
                sh 'touch build/computer.txt'
                sh 'echo "Mainboard" >> build/computer.txt'
            }
        }
    }
}
```

`sh 'echo "Mainboard" >> build/computer.txt'` is the command to append the text *Mainboard* to the file *computer.txt* in the *build* dorectory.

Save and build now.

![Imgur](https://imgur.com/4GWe9mB.png)

There is no Jenkins output to the build regarding the printing of *Mainboard*. 

Configure build and add `sh 'cat build/computer.txt'`

The build failed because:

![Imgur](https://imgur.com/p3zqH0p.png)

Configure.

Change mkdir line to `sh 'mkdir -p build'`. `-p` creates the directory and any necessary parent directories.

Build is successful. 

![Imgur](https://i.imgur.com/o7Ae5yJ.png)

Mainboard appears twice. Strange. The forst occurence is from the first execution, the second is from the recent execution.

Some more commands have been added to the configuration:

```
pipeline {
    agent any

    stages {
        stage('Build') {
            steps {
                echo 'Building a new laptop...'
                sh 'mkdir -p build'
                sh 'touch build/computer.txt'
                sh 'echo "Mainboard" >> build/computer.txt'
                sh 'cat build/computer.txt'
                sh 'echo "Display" >> build/computer.txt'
                sh 'cat build/computer.txt'
                sh 'echo "Keyboard" >> build/computer.txt'
                sh 'cat build/computer.txt'
            }
        }
    }
}

```

Here is the output:

![Imgur](https://i.imgur.com/EcMHjvB.png)

### Reason for duplicate outputs

Jenkins has a Workspace where it runs the build and saves the build result.

![Imgur](https://i.imgur.com/1wSoOu9.png)

One way to combat the duplicates is to configure the pipeline to remove the directory when starting a new build. This isn't always the best approach as there could be multiple files/directories in a build, so other builds could be affected.

There is an instruction called clean workspace. This would be a post-build action.

In a post-build action we can specify actions to take place for example if a build fails or if it's successful.

To clean the workspace after every build:

```
pipeline {
    agent any

    stages {
        stage('Build') {
            steps {
                echo 'Building a new laptop...'
                sh 'mkdir -p build'
                sh 'touch build/computer.txt'
                sh 'echo "Mainboard" >> build/computer.txt'
                sh 'cat build/computer.txt'
                sh 'echo "Display" >> build/computer.txt'
                sh 'cat build/computer.txt'
                sh 'echo "Keyboard" >> build/computer.txt'
                sh 'cat build/computer.txt'
            }
        }
    }
    post {
        always {
            cleanWs()
        }
    }
}

```

The immediate output still has the duplicate displays, but has the clean workspace action:

![Imgur](https://i.imgur.com/RBK9kl2.png)

Here is the following build:

![Imgur](https://i.imgur.com/OvrCiDb.png)

## Artifacts

The file created in `build/` is called an Artifact. When we clean the workspace at the end of each build, we want to keep the artifact.

This instruction can be added to the configuration.

```
pipeline {
    agent any

    stages {
        stage('Build') {
            steps {
                echo 'Building a new laptop...'
                sh 'mkdir -p build'
                sh 'touch build/computer.txt'
                sh 'echo "Mainboard" >> build/computer.txt'
                sh 'cat build/computer.txt'
                sh 'echo "Display" >> build/computer.txt'
                sh 'cat build/computer.txt'
                sh 'echo "Keyboard" >> build/computer.txt'
                sh 'cat build/computer.txt'
            }
        }
    }
    post {
        success {
            archiveArtifacts artifacts: 'build/**'
        }
        always {
            cleanWs()
        }
    }
}

```

`build/**` denotes everything in the build directory.

Running this produces an error with this output:

![Imgur](https://i.imgur.com/JhkCaDr.png)