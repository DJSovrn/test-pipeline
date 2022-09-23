def junitUtils

pipeline {
  agent { node { label 'build-label-webtech' } }
  
  environment {
    USER_CREDENTIALS = credentials('artifact-user')
    PIPELINE_LABEL = "1.0.${env.BUILD_NUMBER}"
    CONTAINER = 'jnlp-agent-build-node'
    VERSION = "$PIPELINE_LABEL"
    GRADLE_USER_HOME = '/home/jenkins/gradle/.gradle'
  }
  
stages {
    stage('Build artifacts') {
      steps {
          echo "Just echoing on the agent ho-ho-ho-ho"
      }
    }
    stage('Load Shared Libraries') {
      steps {
        script {
          def LIB = load('groovy/sharedLibraryLoader.groovy').loadSharedLibrary()
          junitUtils = LIB.com.sovrn.jenkins.junit.JUnitUtils.new()
        }
      }
    }
  stage ('Gradle setup') {
      steps {
        container("$CONTAINER") {
          timestamps {
            script {
              sh '''
                echo "org.gradle.caching = true" >> gradle.properties
                echo "org.gradle.daemon = false" >> gradle.properties
                echo "artifactory_user = $USER_CREDENTIALS_USR" >> gradle.properties
                echo "artifactory_password = $USER_CREDENTIALS_PSW" >> gradle.properties
              '''
            }
          }
        }
      }
    }
  stage('Gradle testing') {
      steps {
        container("$CONTAINER") {
          timestamps {
            lock('exchange-shared-gradle-cache') {
              withDockerRegistry([credentialsId: "artifact-user", url: "https://sovrn-docker.jfrog.io"]) {
                script {
                  junitUtils.withReport {
                    sh '''
                      ./gradlew pullMySqlImage
                    '''
                  }
                }
              }
            }
          }
        }
      }
    }
}
}
