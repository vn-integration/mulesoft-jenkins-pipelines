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
          // echo "~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~"
          // sh 'java -version'
          // echo "~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~"

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
          ## MULE_VERSION = ${MULE_VERSION}
          ## APPLICATION_NAME = ${APPLICATION_NAME}
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

    stage('Build and test application') {
      steps {
        script {
          MAVEN_PARAMETERS = "--no-transfer-progress -batch-mode --errors --show-version --update-snapshots -DskipMunitTests"
          sh "mvn ${MAVEN_PARAMETERS} clean package"
        }
      }
    }

    stage('Sonarqube analysis') {
      steps {
        withSonarQubeEnv('Sonarqube local') {
          sh "mvn sonar:sonar -Dsonar.projectKey=${APPLICATION_NAME} -Dsonar.sources=."
        }
      }
    }

    stage('Upload artifact') {
      steps {
        // script {
        //   MAVEN_PARAMETERS = "--no-transfer-progress -batch-mode --errors --show-version --update-snapshots -DskipMunitTests"
        //   // MAVEN_NEXUS_PARAMETERS = "-DrepositoryId=nexus-maia -Durl=$common_properties.NEXUS_BASE_URL/repository/$repo -Dfile=target/$JAR_FILE -DpomFile=pom.xml"
        //   MAVEN_NEXUS_PARAMETERS = "-Durl=http://nexus:8081/repository/maven-snapshots -Dfile=target/${APPLICATION_NAME}.jar"
        //   sh "mvn ${MAVEN_PARAMETERS} deploy:deploy-file ${MAVEN_NEXUS_PARAMETERS}"

        //   // mvn --no-transfer-progress -batch-mode --errors --show-version --update-snapshots -D"skipMunitTests" deploy:deploy-file -D"url=http://localhost:18081/repository/mulesoft-snapshots/" -D"file=target/mule-application-template-1.0.0-mule-application.jar" -D"pomFile=pom.xml" -D"repo.id=mulesoft-snapshots" -D"repo.login=jenkins" -D"repo.pwd=jenkins"
        //   // mvn --no-transfer-progress -batch-mode --errors --show-version --update-snapshots -D"skipMunitTests" deploy:deploy-file -D"url=http://localhost:18081/repository/mulesoft-snapshots/" -D"file=target/mule-application-template-1.0.0-mule-application.jar" -D"pomFile=pom.xml"
        // }
        echo 'Upload artifact to package repository'
      }
    }

    stage('Notify result') {
      steps {
        echo 'Notify build result'
      }
    }

  }

}
