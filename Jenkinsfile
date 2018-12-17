#!/usr/bin/env groovy Jenkinsfile

library 'JenkinsSharedLibraries'

def clearDocker(String imagename) {
    if(sh (script: "docker ps -a|grep "+imagename, returnStatus: true)  == 0){
        sh "docker rm -f "+imagename
    }

    if(sh (script: "docker images|grep "+imagename, returnStatus: true)  == 0){
        sh "docker rmi "+imagename
    }
}

pipeline {
    agent {
        node {
            label 'slave-1'
        }
    }
    environment {
        DOCKER_LOGIN     = credentials('docker_login')
    }
    triggers {
      githubPush()
    }
    stages {

        stage('docker-login') {
            when {
                expression { ciRelease action: 'check' }
            }
            steps {
                sh "docker login $DOCKER_LOGIN"
            }
        }

        stage('build-image:aspnetcore2.2') {
            stages {
                stage('pull-latest-image:aspnetcore2.2') {
                    when {
                        expression { ciRelease action: 'check' }
                    }
                    steps {
                        sh "docker pull microsoft/dotnet:2.2.0-aspnetcore-runtime"
                    }
                }

                stage('build-image:aspnetcore2.2') {
                    steps {
                        sh '''
                        cd docker/aspnetcore2.2;
                        chmod +x build.sh;
                        ./build.sh
                        '''
                    }
                }

                stage('push-image:aspnetcore2.2') {
                    when {
                        expression { ciRelease action: 'check' }
                    }
                    steps {
                        sh "docker push stulzq/dotnet:2.2.0-aspnetcore-runtime-with-image"
                    }
                }
            }

            options {
                timeout(time: 30, unit: 'MINUTES')
            }
        }

        stage('test-image:aspnetcore2.2') {
            stages {
                stage('build-test-image:aspnetcore2.2') {
                    steps {
                        clearDocker('awesomedotnetcoreimagehello')
                        sh '''
                        cd src/awesome-dotnetcore-image-hello/awesome-dotnetcore-image-hello;
                        chmod +x build-image.sh;
                        ./build-image.sh
                        '''
                    }
                }
                stage('run-test:aspnetcore2.2') {
                    steps {
                        sh "docker run -d --rm -p 5009:80 --name awesomedotnetcoreimagehello awesomedotnetcoreimagehello"
                        sleep(time:10,unit:"SECONDS")
                        sh "curl http://localhost:5009/api/values"
                    }
                }
                stage('clear-test:aspnetcore2.2') {
                    steps {
                        clearDocker('awesomedotnetcoreimagehello')
                    }
                }
            }
        }

    }
    
    post { 
        always { 
            // clear command history
            sh "history -c"
            sh "echo > $HOME/.bash_history"

            
            sh "docker logout" 

            // cleanr workspace
            deleteDir() 
        }
    }
}
