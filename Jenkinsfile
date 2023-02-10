#! /usr/bin/env groovy

pipeline {

  agent any
  environment {
    SPECTRAL_DSN = credentials('spectral-dsn')
  }
  stages {
    stage('install Spectral') {
      steps {
        sh "curl -L 'https://spectral-eu.checkpoint.com/latest/x/sh?dsn=$SPECTRAL_DSN' | sh"
      }
    }
    stage('scan for issues') {
      steps {
        sh "$HOME/.spectral/spectral scan --ok  --include-tags base,audit"
      }
    }
    stage('Create Container Image') {
      steps {
        echo 'Create Container Image..'
        
        script {
          openshift.withCluster() {
            openshift.withProject("jenkins") {
                def buildConfigExists = openshift.selector("bc", "codelikethewind").exists()

                if(!buildConfigExists){
                    openshift.newBuild("--name=codelikethewind", "--docker-image=nginx:latest")
                }

                openshift.selector("bc", "codelikethewind").startBuild()

            }

          }
        }
      }
    }
  }
}
