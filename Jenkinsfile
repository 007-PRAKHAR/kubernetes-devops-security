pipeline {
  environment { 
        registry = 'https://hub.docker.com/r/prakhar0012/numeric-app' 
        registryCredential = 'docker-hub' 
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
              docker.withRegistry(registry, registryCredential){
                sh 'printenv'
                sh 'docker build -t prakhar0012/numeric-app:""$GIT_COMMIT"" . '
                sh 'docker push prakhar0012/numeric-app:""$GIT_COMMIT""'
              }
            }
        }
    }
}
