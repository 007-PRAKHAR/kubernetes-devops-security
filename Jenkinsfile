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
    stage('Docker build and push') {
            steps {
                sh 'docker build -t prakhar0012/numeric-app:$BUILD_NUMBER . '
                sh 'echo $DOCKER_CRED_PSW | docker login -u $DOCKER_CRED_USR --password-stdin'
                sh 'docker push prakhar0012/numeric-app:$BUILD_NUMBER'
      }
    }
    stage('kubernetes deployment'){
            steps {
              withKubeConfig([credentialsId: 'kubeconfig']) {
                sh "sed -i 's#replace#prakhar0012/numeric-app:$BUILD_NUMBER#g' k8s_deployment_service.yaml"
                sh "kubectl apply -f k8s_deployment_service.yaml"
              }
            }
          }
        }
post {
  always {
    sh 'docker logout'
  }
}
}
