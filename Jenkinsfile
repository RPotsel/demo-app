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
        EXAMPLE_KEY = credentials('KUBECONFIG_STAGE')
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
                // PushImageToRegistry('./gateway', 'demo-app-gateway:latest')
                // PushImageToRegistry('./gowebserver', 'demo-app-gowebserver:latest')

                // withKubeConfig([credentialsId: 'KUBECONFIG_STAGE', namespace: 'demo-app']) {
                //     sh "helm upgrade demo-app helm/demo-app/ --reuse-values --set ingress.host=demo.stage.pinbit.ru --set gowebserver.env[0].name=WORKSPACE --set gowebserver.env[0].value=STAGE1"
                //     sh "kubectl get all -A"
                // }

                // withCredentials([file(credentialsId: 'KUBECONFIG_STAGE', variable: 'config')]) {
                //     sh """
                //         export KUBECONFIG=\${config}
                //         kubectl get all -A
                //         helm upgrade demo-app helm/demo-app/ --reuse-values --set ingress.host=demo.stage.pinbit.ru --set gowebserver.env[0].name=WORKSPACE --set gowebserver.env[0].value=STAGE1 -n demo-app
                //     """
                // }

                // withCredentials([string(credentialsId: 'KUBECONFIG_STAGE', variable: 'SECRET')]) { //set SECRET with the credential content
                //     echo "My secret text is '${SECRET}'"
                // }

                sh('echo ${EXAMPLE_KEY}')
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
