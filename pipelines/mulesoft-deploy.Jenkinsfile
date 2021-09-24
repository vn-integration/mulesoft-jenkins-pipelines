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
          ## CLOUDHUB_SKIP_VERIFY_DEPLOYMENT = ${CLOUDHUB_SKIP_VERIFY_DEPLOYMENT}
          ##
          ## #################################################################################################
          """
        }
      }
    }

    stage('Retrieve artifact') {
      steps {
        // TODO: Retrieve from Nexus
        git credentialsId: 'GITEA_CREDENTIALS', branch: APPLICATION_BRANCH, url: APPLICATION_REPOSITORY
        echo 'Download artifact from package repository'
      }
    }

    stage('Deploy to Cloudhub') {
      steps {
        script {
          MAVEN_PARAMETERS = "--no-transfer-progress -batch-mode --errors --show-version --update-snapshots"
          DEPLOYMENT_PARAMETERS = "-Dcloudhub.username=${ANYPOINT_USERNAME} -Dcloudhub.password=${ANYPOINT_PASSWORD} -Dcloudhub.environment=${CLOUDHUB_ENVIRONMENT} -Dcloudhub.businessGroupId=${BUSINESS_GROUP_ID} -Dapp.workers=1 -Dapp.workerType=MICRO -Dcloudhub.skipDeploymentVerification=${CLOUDHUB_SKIP_VERIFY_DEPLOYMENT} -Dproject.name=${APPLICATION_NAME}"
          sh "mvn ${MAVEN_PARAMETERS} ${DEPLOYMENT_PARAMETERS} deploy -DmuleDeploy -DskipMunitTests -Dtest.runCoverage=false"
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
