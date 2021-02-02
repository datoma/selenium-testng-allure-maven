pipeline {
  agent {
    docker {
      image 'maven:3-alpine'
      args '-v /root/.m2:/root/.m2'
    }
  }
  stages {
    stage('Build') {
      steps {
        sh 'mvn clean verify -P parallelPlugin,nogrid -Dthreads=3 -Dbrowser=chrome'
      }
    }
    stage('parallel Test') {
      failFast true
      parallel {
        stage('Test') {
          steps {
            sh 'mvn test'
          }
          post {
            always {
              junit 'target/surefire-reports/*.xml'
            }
          }
        }
        stage('Sonarqube') {
          environment {
            scannerHome = tool 'SonarScanner'
          }
          steps {
            withSonarQubeEnv('Sonar') {
              sh "${scannerHome}/bin/sonar-scanner"
            }
            timeout(time: 10, unit: 'MINUTES') {
              waitForQualityGate abortPipeline: true
            }
          }
        }
        stage('Publish Test Coverage Report') {
          steps {
            step([$class: 'JacocoPublisher',
              execPattern: '**/build/jacoco/*.exec',
              classPattern: '**/build/classes',
              sourcePattern: 'src/main/java',
              exclusionPattern: 'src/test*'
            ])
          }
        }
      }
    }
    stage('Deliver to local maven repo') {
      steps {
        sh './jenkins/scripts/deliver.sh'
      }
    }
  }
  post {
    always {
      archiveArtifacts artifacts: 'build/libs/**/*.jar', fingerprint: true
      junit 'build/reports/**/*.xml'
    }
  }
}
