pipeline {
    agent any
    stages {
            stage ("Prepare Docker"){
                steps {
                        sh 'docker rm --force crud'
                        sh 'docker rm --force tests'
                        sh 'docker network ls|grep external-api > /dev/null || docker network create --driver bridge external-api'
                }
            }
            stage('Build and run CRUD app') {
                steps {
                        git credentialsId: '0bade186-02c7-4982-a481-adc7f91286d8', url: 'https://github.com/YaroslavBrek/crud-app.git'
                        sh 'docker build -t app .'
                        sh 'docker run \
                                -d \
                                --network "external-api" \
                                --name "crud" \
                                -p 9000:9000 \
                                app'
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
                        sh 'docker build -t tests .'
                        sh 'docker run \
                                --name "tests" \
                                --network "external-api" \
                                -p 9001:9000 \
                                tests'
            }
        }
    }
}