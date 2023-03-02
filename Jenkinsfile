pipeline {
    agent any
    environment {
        TEST_GROUP='api'
        ENV_URL='crud'
        ENV_PORT='9000'
        DOCKER_NETWORK='external-api'
    }
    stages {
            stage ("Prepare Docker"){
                steps {
                    script {
                        sh "echo 'Build number is ${currentBuild.number}'"
                        sh "docker rm --force crud"
                        sh "docker rm --force tests"
                        sh "docker network ls|grep ${env.DOCKER_NETWORK} > /dev/null || docker network create --driver bridge ${env.DOCKER_NETWORK}"
                    }
                }
            }
            stage('Build and run CRUD app') {
                steps {
                    git 'https://github.com/YaroslavBrek/crud-app.git'
                    script {
                        sh "docker build -t app ."
                        sh "docker run \
                                -d \
                                --network '${env.DOCKER_NETWORK}' \
                                --name 'crud' \
                                -p ${env.ENV_PORT}:${env.ENV_PORT} \
                                app"
                            }                        
                }
            }
            stage ("Wait until app is up") {
                 steps {
                    script {
                        sh "sleep 10"
                    }
                 }
            }
            stage('Run tests against CRUD app') {
                steps {
                    git 'https://github.com/YaroslavBrek/api-tests.git'
                    script {
                        sh "docker build -t tests --build-arg envUrl=${env.ENV_URL} --build-arg envPort=${env.ENV_PORT} --build-arg testGroup=${env.TEST_GROUP} ."
                        sh "docker run \
                                --name 'tests' \
                                --network '${env.DOCKER_NETWORK}' \
                                -v allure-results:/var/target/allure-results \
                                -p 9001:${env.ENV_PORT} \
                                tests"
                        }
            }
        }
    }
}