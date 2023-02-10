#! /usr/bin/env groovy

pipeline {

  agent any
  stages {

    stage('Create Container Image') {
      steps {
        echo 'Create Container Image..'
        
        script {
          openshift.withCluster() {
            openshift.withProject("jenkins") {
                def buildConfigExists = openshift.selector("bc", "codelikethewind").exists()

                if(!buildConfigExists){
                    openshift.newBuild("--name=codelikethewind", "--docker-image=registry.redhat.io/jboss-eap-7/eap74-openjdk8-openshift-rhel7", "--binary")
                }

                openshift.selector("bc", "codelikethewind").startBuild()

            }

          }
        }
      }
    }
    stage('Deploy') {
      steps {
        echo 'Deploying....'
        script {
          openshift.withCluster() {
            openshift.withProject("jenkins") {

              def deployment = openshift.selector("dc", "codelikethewind")

              if(!deployment.exists()){
                openshift.newApp('codelikethewind', "--as-deployment-config").narrow('svc').expose()
              }

              timeout(5) { 
                openshift.selector("dc", "codelikethewind").related('pods').untilEach(1) {
                  return (it.object().status.phase == "Running")
                  }
                }

            }

          }
        }
      }
    }
  }
}
