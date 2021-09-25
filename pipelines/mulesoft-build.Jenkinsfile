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

          SKIP_TESTS = env.SKIP_TESTS ?: false
          SKIP_COVERAGE = env.SKIP_COVERAGE ?: false
          APPLICATION_BRANCH = env.APPLICATION_BRANCH ?: "development"

          echo """
          ## Pipeline parameters #############################################################################
          ##
          ## ENV = ${ENV}
          ## APPLICATION_NAME = ${APPLICATION_NAME}
          ## APPLICATION_REPOSITORY = ${APPLICATION_REPOSITORY}
          ## APPLICATION_BRANCH = ${APPLICATION_BRANCH}
          ## SKIP_TESTS = ${SKIP_TESTS}
          ## SKIP_COVERAGE = ${SKIP_COVERAGE}
          ##
          ## #################################################################################################
          """
        }
      }
    }

    stage('Checkout code') {
      steps {
        git credentialsId: 'GITEA_CREDENTIALS', branch: APPLICATION_BRANCH, url: APPLICATION_REPOSITORY
      }
    }

    stage('Build and test application') {
      steps {
        script {
          MAVEN_PARAMETERS = "--no-transfer-progress -batch-mode --errors --update-snapshots"
          MAVEN_PARAMETERS = MAVEN_PARAMETERS + (SKIP_TESTS ? " -DskipMunitTests" : "")
          MAVEN_PARAMETERS = MAVEN_PARAMETERS + (SKIP_COVERAGE ? " -Dtest.runCoverage=false" : " -Dtest.runCoverage=true")

          sh "mvn ${MAVEN_PARAMETERS} clean package"
        }
      }
    }

    stage('SonarQube analysis') {
      steps {
        script {
          MAVEN_PARAMETERS = "--no-transfer-progress -batch-mode --errors --update-snapshots"
        }
        withSonarQubeEnv('sonar-mule') {
          sh "mvn ${MAVEN_PARAMETERS} sonar:sonar -Dsonar.projectKey=${APPLICATION_NAME} -Dsonar.sources=."
        }
      }
    }

    stage('Upload artifact') {
      steps {
        script {
          MAVEN_PARAMETERS = "--no-transfer-progress -batch-mode --errors --update-snapshots"
          MAVEN_PARAMETERS = MAVEN_PARAMETERS + " -DskipMunitTests -Dtest.runCoverage=false"

          APP_VERSION = sh (script: "mvn help:evaluate -Dexpression=project.version -q -DforceStdout", returnStdout: true)
          APP_ARTIFACT_ID = sh (script: "mvn help:evaluate -Dexpression=project.artifactId -q -DforceStdout", returnStdout: true)
          MAVEN_NEXUS_PARAMETERS = "-Durl=http://nexus:8081/repository/mule-poc-snapshots/ -Dfile=target/${APPLICATION_NAME}-${APP_VERSION}-mule-application.jar -DpomFile=pom.xml"

          sh "mvn ${MAVEN_PARAMETERS} deploy:deploy-file ${MAVEN_NEXUS_PARAMETERS}"
        }
        echo 'Upload artifact to package repository'
      }
    }

    // stage('Deploy to Cloudhub') {
    //   steps {
    //     script {
    //       MAVEN_PARAMETERS = "--no-transfer-progress -batch-mode --errors --update-snapshots"
    //       DEPLOYMENT_PARAMETERS = "-Dcloudhub.username=${ANYPOINT_USERNAME} -Dcloudhub.password=${ANYPOINT_PASSWORD} -Dcloudhub.environment=${CLOUDHUB_ENVIRONMENT} -Dcloudhub.businessGroupId=${BUSINESS_GROUP_ID} -Dapp.workers=1 -Dapp.workerType=MICRO -Dcloudhub.skipDeploymentVerification=false -Dproject.name=${APPLICATION_NAME}"
    //       sh "mvn ${MAVEN_PARAMETERS} ${DEPLOYMENT_PARAMETERS} deploy -DmuleDeploy -DskipMunitTests -Dtest.runCoverage=false"
    //     }
    //   }
    // }

    // stage ('Call deploy job') {
    //   steps {
    //     build job: "${APPLICATION_NAME}-deploy", parameters: [[
    //       $class: 'StringParameterValue', name: 'env', value: 'DEV'
    //     ]]
    //   }
    // }

    stage('Notify result') {
      steps {
        echo 'Notify build result'
      }
    }
  }

  post {
    always {
      junit 'target/surefire-reports/*.xml'

      publishHTML (target : [
        allowMissing: false,
        alwaysLinkToLastBuild: true,
        keepAll: true,
        reportDir: 'target/site/munit/coverage',
        reportFiles: 'summary.html',
        reportName: 'Coverage report',
        reportTitles: 'Coverage report'
      ])
    }
  }
}