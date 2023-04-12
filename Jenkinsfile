#!/usr/bin/env groovy

properties([disableConcurrentBuilds()])

pipeline {
    agent { label 'built-in' }
    options {
        skipStagesAfterUnstable()
        skipDefaultCheckout()
        buildDiscarder(logRotator(numToKeepStr: '10', artifactNumToKeepStr: '10'))
    }
    environment {
        IMAGE_BASE = 'rpot'
        DEMO_APP_NAMESPACE = 'demo-app'
    }
    stages {
        stage('Checkout') {
            steps {
                PrintStage()
                checkout scm

                sh "ls -ltra"
                echo sh(script: 'env|sort', returnStdout: true)
            }
        }
        stage('Deploy to STAGE') {
            when {
                branch 'main'
            }
            steps {
                PrintStage()
                PushImageToRegistry('./gateway', 'demo-app-gateway:latest')
                PushImageToRegistry('./gowebserver', 'demo-app-gowebserver:latest')

                withKubeConfig([credentialsId: 'STAGE_JENKINS_ROBOT_TOKEN', 
                                serverUrl: "${STAGE_CLUSTER_URL}", 
                                namespace: "${DEMO_APP_NAMESPACE}"]) {
                    sh """
                        helm upgrade demo-app helm/demo-app/ --reuse-values \
                            --set ingress.host=demo.stage.pinbit.ru \
                            --set gowebserver.env[0].name=WORKSPACE \
                            --set gowebserver.env[0].value=STAGE
                        kubectl get all
                    """
                }
            }
        }
        stage('Deploy to PROD') {
            when {
                tag "v.*"
            }
            steps {
                PrintStage()
                PushImageToRegistry('./gateway', "demo-app-gateway:${env.TAG_NAME}")
                PushImageToRegistry('./gowebserver', "demo-app-gowebserver:${env.TAG_NAME}")

                withKubeConfig([credentialsId: 'PROD_JENKINS_ROBOT_TOKEN', 
                                serverUrl: "${PROD_CLUSTER_URL}", 
                                namespace: "${DEMO_APP_NAMESPACE}"]) {
                    sh """
                        helm upgrade demo-app helm/demo-app/ --reuse-values \
                            --set ingress.host=demo.prod.pinbit.ru \
                            --set gowebserver.env[0].name=WORKSPACE \
                            --set gowebserver.env[0].value=PROD
                        kubectl get all
                    """
                }
            }
        }
    }
}

void PrintStage(String text="") {
    text=="" ? println('* '*10 + env.STAGE_NAME.toUpperCase() + " *"*10) : println(text)
}

void PushImageToRegistry(String dockerFilePath=".", String imageName) {
    def dockerImage = docker.build("${env.IMAGE_BASE}/${imageName}", "${dockerFilePath}")
    sh "docker image ls"
    docker.withRegistry('','DOCKERHUB_USER') {
        dockerImage.push()
    }
}
