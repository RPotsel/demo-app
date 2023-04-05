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
