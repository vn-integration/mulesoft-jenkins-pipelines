pipeline {

  agent any

  tools {
    maven 'M3'
  }

  environment {
  }

  stages {

    stage('Setup environment') {
      steps {
        script {
          if (ENV == null) {
             error "ENV parameter is null"
          }

          if (APPLICATION_NAME == null) {
             error "APPLICATION_NAME parameter is null"
          }

          echo """
          ## Pipeline parameters #############################################################################
          ##
          ## ENV = ${ENV}
          ## APPLICATION_NAME = ${APPLICATION_NAME}
          ## MULE_VERSION = ${MULE_VERSION}
          ## APPLICATION_REPOSITORY = ${APPLICATION_REPOSITORY}
          ## APPLICATION_BRANCH = ${APPLICATION_BRANCH}
          ##
          ## #################################################################################################
          """
        }
      }
    }

    stage('Checkout code') {
      steps {
        git branch: APPLICATION_BRANCH, url: APPLICATION_REPOSITORY
      }
    }

    stage('Build application') {
      steps {
        script {
          MAVEN_PARAMETERS = "--no-transfer-progress -batch-mode --errors --show-version --update-snapshots -DskipMunitTests"
          sh "mvn ${MAVEN_PARAMETERS} clean package"
        }
      }
    }

    stage('Upload package') {
      steps {
        echo 'Upload artifact to package repository'
      }
    }

  }

}
