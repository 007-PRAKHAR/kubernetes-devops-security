pipeline {

  environment{
    DOCKER_CRED = credentials('docker-hub')
  } 
  
  agent any

  stages {
      stage('Build Artifact') {
            steps {
              sh "mvn clean package -DskipTests=true"
              archive 'target/*.jar'
            }
        }

    stage('Unit Testing - JUnit and JaCoco') {
            steps {
              sh "mvn test"
            }
            post {
              always{
                junit 'target/surefire-reports/*.xml'
                jacoco (
                  execPattern: 'target/jacoco.exec'
                  )
              }
            }
        }
    stage('Docker build') {
            steps {
                sh 'docker build -t prakhar0012/numeric-app:$BUILD_NUMBER . '
              }
            }
    stage ('Login to docker'){
      steps{
        sh 'echo $DOCKER_CRED_PSW | docker login -u $DOCKER_CRED_USR --password-stdin'
      }
    }
    stage ('Push Docker image'){
      steps{
        sh 'docker push prakhar0012/numeric-app:$BUILD_NUMBER'
      }
    }
        }
post {
  always {
    sh 'docker logout'
  }
}
}
