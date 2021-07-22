pipeline {

  agent any

  tools {
    maven 'M3'
  }

  environment {
    BUILDER = "jenkins"
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
          ## CLOUDHUB_ENVIRONMENT = ${CLOUDHUB_ENVIRONMENT}
          ## CLOUDHUB_WORKER_SIZE = ${CLOUDHUB_WORKER_SIZE}
          ## BUSINESS_GROUP_ID = ${BUSINESS_GROUP_ID}
          ## CLOUDHUB_VERIFY_DEPLOYMENT = ${CLOUDHUB_VERIFY_DEPLOYMENT}
          ##
          ## #################################################################################################
          """
        }
      }
    }

    // stage('Checkout code') {
    //   steps {
    //     git branch: APPLICATION_BRANCH, url: APPLICATION_REPOSITORY
    //   }
    // }

    // stage('Build and test application') {
    //   steps {
    //     script {
    //       MAVEN_PARAMETERS = "--no-transfer-progress -batch-mode --errors --show-version --update-snapshots -DskipMunitTests"

    //       sh "mvn ${MAVEN_PARAMETERS} clean package"
    //     }
    //   }
    // }

    // stage('Sonarqube analysis') {
    //   steps {
    //     withSonarQubeEnv('Sonarqube local') {
    //       MAVEN_SONAR_PARAMETERS = "-Dsonar.projectKey=${APPLICATION_NAME} -Dsonar.sources=."

    //       sh "mvn sonar:sonar ${MAVEN_SONAR_PARAMETERS}"
    //     }
    //   }
    // }

    stage('Retrieve artifact') {
      steps {
        echo 'Download artifact from package repository'
      }
    }

    stage('Deploy to Cloudhub') {
      steps {
        script {
          MAVEN_PARAMETERS = "--no-transfer-progress -batch-mode --errors --show-version --update-snapshots -DskipMunitTests"
          MAVEN_DEPLOYMENT_PARAMETERS = "-Dmule.version=${MULE_VERSION} -Danypoint.username=${ANYPOINT_USERNAME} -Danypoint.password=${ANYPOINT_PASSWORD} -Dcloudhub.applicationName=${APPLICATION_NAME} -Dcloudhub.environment=${CLOUDHUB_ENVIRONMENT} -Dcloudhub.worker=${CLOUDHUB_WORKER_SIZE} -Dcloudhub.businessGroup=${BUSINESS_GROUP_ID} -Dcloudhub.verifyDeployment=${CLOUDHUB_VERIFY_DEPLOYMENT}"

          sh "mvn ${MAVEN_PARAMETERS} deploy -DmuleDeploy ${MAVEN_DEPLOYMENT_PARAMETERS}"
        }
      }
    }

    stage('Notify result') {
      steps {
        echo 'Notify deploy result'
      }
    }

  }

}
