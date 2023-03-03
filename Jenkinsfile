pipeline {
    agent any
    environment {
        TEST_GROUP='api'
        ENV_URL='crud'
        ENV_PORT='9000'
        ALLURE_REPORT_PORT='9090'
        DOCKER_NETWORK='external-api'
        BUILD_NUMBER="${currentBuild.number}"
        JENKINS_CONTAINER_NAME='jenkins_in_docker'
        TESTS_CONTAINER_NAME='tests'
        APP_CONTAINER_NAME='crud'
        ENV_WORKSPACE="${env.WORKSPACE}"
    }
    stages {
            stage ("Prepare Docker"){
                steps {
                    script {
                        sh "echo 'Build number is ${env.BUILD_NUMBER}'"
                        sh "echo ${env.ENV_WORKSPACE}"
                        sh "docker rm --force ${env.APP_CONTAINER_NAME}"
                        sh "docker rm --force ${env.TESTS_CONTAINER_NAME}"
                        sh "docker network ls|grep ${env.DOCKER_NETWORK} > /dev/null || docker network create --driver bridge ${env.DOCKER_NETWORK}"
                    }
                }
            }
            stage('Build and run CRUD app') {
                steps {
                    git 'https://github.com/YaroslavBrek/crud-app.git'
                    script {
                        sh "docker build -t ${env.APP_CONTAINER_NAME} ."
                        sh "docker run \
                                -d \
                                --network '${env.DOCKER_NETWORK}' \
                                --name '${env.APP_CONTAINER_NAME}' \
                                -p ${env.ENV_PORT}:${env.ENV_PORT} \
                                ${env.APP_CONTAINER_NAME}"
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
                        sh "docker build -t ${env.TESTS_CONTAINER_NAME} --build-arg envUrl=${env.ENV_URL} --build-arg envPort=${env.ENV_PORT} --build-arg testGroup=${env.TEST_GROUP} ."
                        sh "docker run -d -ti --rm \
                                --name '${env.TESTS_CONTAINER_NAME}' \
                                --network '${env.DOCKER_NETWORK}' \
                                --volumes-from ${env.JENKINS_CONTAINER_NAME} \
                                -p ${env.ALLURE_REPORT_PORT}:${env.ALLURE_REPORT_PORT} \
                                ${env.TESTS_CONTAINER_NAME}"
                    }
                }
            }
            stage ("Copy tests results") {
                   steps {
                       script {
                            sh "docker exec tests cp -R target/allure-results/ ${env.ENV_WORKSPACE}/target/allure-results"
                       }
                   }
            }
//             stage('Generate Allure Report') {
//                 steps {
//                     script {
//                         allure([
//                                 includeProperties: false,
//                                 jdk: '',
//                                 properties: [],
//                                 reportBuildPolicy: 'ALWAYS',
//                                 results: [[path: 'target/allure-results']]
//                         ])
//                     }
//                 }
//             }
    }
}