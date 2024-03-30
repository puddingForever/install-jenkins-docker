# Composing Jenkins with Docker Container 

This repository contains a Docker Compose configuration for a quick installation of Jenkins. This setup is not intended for production systems.

Credits: This approach is mostly based on the [offical instructions](https://www.jenkins.io/doc/book/installing/docker/) but takes advantage of Docker Dompose (by using a `docker-compose.yml` file) to reduce the number of steps needed to get Jenkins up and running.

## Step 1

Install Docker locally (probably using Docker Desktop is the easiest approach).

## Step 2

Clone this repository or download it's contents. 

## Step 2

Open a terminal window in the same directory where the `Dockerfile` from this repository is located. Build the Jenkins Docker image:

```
docker build -t my-jenkins .
```

## Step 3

Start Jenkins:

```
docker compose up -d
```

## Step 4

Open Jenkins by going to: [http://localhost:8080/](http://localhost:8080/) and finish the installation process.

## Step 5

If you wish to stop Jenkins and get back to it later, run:

```
docker compose down
```

If you wish to start Jenkins again later, just run the same comand from Step 3.


# Removing Jenkins

Once you are done playing with Jenkins maybe it is time to clean things up.

Run the following comand to terminate Jenkins and to remove all volumes and images used:

```
docker compose down --volumes --rmi all 
```


# Creating artifacts 

in the configure menu, go to pipepline and type <br>

archiveArtifacts artifacts : 'path'

```
pipeline {
    agent any

    stages {
        stage('Build') {
            steps {
                cleanWs()
                echo 'Building a new laptop'
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
    
    post{
        success{
            archiveArtifacts artifacts: 'build/**'
        }
    }
}

```
Post will activated after the stages are completed successfully. 

# Optimizing shell 
We can optimize duplicates by using ''' 
```
            steps {
                cleanWs()
                echo 'Building a new laptop'
                sh '''
                mkdir -p build
                touch build/computer.txt
                echo "Mainboard" >> build/computer.txt
                cat build/computer.txt
                echo "Display" >> build/computer.txt
                cat build/computer.txt
                echo "Keyboard" >> build/computer.txt
                cat build/computer.txt
                '''
```
Because sh command executed once, it's faster  <br>
![image](https://github.com/puddingForever/install-jenkins-docker/assets/126591306/343613c2-8b48-47aa-a97f-6b41a48042de)



# Testing stage
we have to include testing stage in every development. just include stage('Test') after build stage 

```
pipeline {
    agent any

    stages {
        stage('Build') {
            steps {
                cleanWs()
                echo 'Building a new laptop'
                sh '''
                mkdir -p build
                touch build/computer.txt
                echo "Mainboard" >> build/computer.txt
                cat build/computer.txt
                echo "Display" >> build/computer.txt
                cat build/computer.txt
                echo "Keyboard" >> build/computer.txt
                cat build/computer.txt
                rm build/computer.txt
                '''
            }
        }
        stage('Test'){
            steps{
                echo 'Testing the new laptop'
                sh 'test -f build/computer.txt'
                
                
            }
        }
    }
    
    post{
        success{
            archiveArtifacts artifacts: 'build/**'
        }
    }
}
```
test -f checks whether the file exits. because of rm command on build stage, we failed, and it will not create artifact  <br>
![image](https://github.com/puddingForever/install-jenkins-docker/assets/126591306/b2dc4b38-8276-439a-8bc3-778fe9a3011e)

# Testing build artifacts
use grep command than cat command, because cat just prints but not stop the execution . <br>
grep command will stop the execution <br>


```
        stage('Test'){
            steps{
                echo 'Testing the new laptop'
                sh '''
                    test -f build/computer.txt
                    grep "Mainboard" build/computer.txt
                '''
                
            }
        }
```

If we failed to search "Mainboard" , the build will be failed at test stage and won't create artifact <br>
![image](https://github.com/puddingForever/install-jenkins-docker/assets/126591306/12d5befe-b673-4e5b-97e3-c34632111720)

# Defining environment variables 
When there is repeated words, we can use global vaeriable inside of environment block 
when it's inside of ''' type $variableName . If it's outside of ''', use "$variableName"
```
pipeline {
    agent any
    
    environment{
        BUILD_FILE_NAME = 'computer.txt'
    }
    

    stages {
        stage('Build') {
            steps {
                cleanWs()
                echo 'Building a new laptop'
                sh '''
                    echo $BUILD_FILE_NAME
                    mkdir -p build
                    echo "Mainboard" >> build/$BUILD_FILE_NAME
                    cat build/$BUILD_FILE_NAME
                    echo "Display" >> build/$BUILD_FILE_NAME
                    cat build/$BUILD_FILE_NAME
                    echo "Keyboard" >> build/$BUILD_FILE_NAME
                    cat build/$BUILD_FILE_NAME
                '''
            }
        }
        stage('Test'){
            steps{
                echo 'Testing the new laptop'
                sh '''
                    test -f build/$BUILD_FILE_NAME
                    grep "Mainboard" build/$BUILD_FILE_NAME
                    grep "Display" build/$BUILD_FILE_NAME
                    grep "Keyboard" build/$BUILD_FILE_NAME
                '''
            }
        }
    }
    
    post{
        success{
            archiveArtifacts artifacts: 'build/**'
        }
    }
}
```




