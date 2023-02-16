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
                        sh "docker rm --force crud"
                        sh "docker rm --force tests"
                        sh "docker network ls|grep ${env.DOCKER_NETWORK} > /dev/null || docker network create --driver bridge ${env.DOCKER_NETWORK}"
                    }
                }
            }
            stage('Build and run CRUD app') {
                steps {
                    git credentialsId: '0bade186-02c7-4982-a481-adc7f91286d8', url: 'https://github.com/YaroslavBrek/crud-app.git'
                    script {
                        sh "docker build -t tests --build-arg envUrl=${env.ENV_URL} --build-arg envPort=${env.ENV_PORT} --build-arg testGroup=${env.TEST_GROUP} ."
                        sh "docker run \
                                -d \
                                --network '${env.DOCKER_NETWORK}' \
                                --name 'crud' \
                                -p 9000:9000 \
                                app"
                            }                        
                }
            }
            stage ("Wait until app is up"){
                 steps {
                        sh 'sleep 20'
                }
            }
            stage('Run tests against CRUD app') {
                steps {
                    git credentialsId: '0bade186-02c7-4982-a481-adc7f91286d8', url: 'https://github.com/YaroslavBrek/api-tests.git'
                    script {
                        sh "docker build -t tests ."
                        sh "docker run \
                                --name 'tests' \
                                --network '${env.DOCKER_NETWORK}' \
                                -p 9001:9000 \
                                tests"
                        }
            }
        }
    }
}