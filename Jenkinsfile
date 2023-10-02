pipeline {

  environment{
    DOCKER_CRED = credentials('docker-hub')
    deploymentName = "devsecops"
    containerName = "devsecops-container"
    serviceName = "devsecops-svc"
    imageName = "siddharth67/numeric-app:${GIT_COMMIT}"
    applicationURL="http://devsecops-demo.eastus.cloudapp.azure.com"
    applicationURI="/increment/99"
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
    stage ('Sonarqube - SAST'){
		steps{
			withSonarQubeEnv ('SonarQube'){
				sh "mvn clean verify sonar:sonar -Dsonar.projectKey=numeric-application -Dsonar.projectName='numeric-application' -Dsonar.host.url=http://devops001.eastus.cloudapp.azure.com:9000 -Dsonar.token=sqp_8f408b54eaa3808ee1dc41b9da95d30321e4ae63"
			}
		}
	}
   stage("Quality Gate") {
            steps {
                timeout(time: 1, unit: 'HOURS') {
                    // Parameter indicates whether to set pipeline to UNSTABLE if Quality Gate fails
                    // true = set pipeline to UNSTABLE, false = don't
                    waitForQualityGate abortPipeline: true
                }
            }
        }
    stage ('Vulnerability Scan - Docker') {
	    steps {
		    parallel (
//			    "Dependency Scan": {
//				    sh "mvn dependency-check:check"
//			    },
			    "Trivy Scan": {
				    sh "bash trivy-docker-image-scan.sh"
			    },
			    "OPA Conftest":{
	 			sh 'docker run --rm -v $(pwd):/project openpolicyagent/conftest test --policy opa-docker-security.rego Dockerfile'
	  		}  
		    )
	    }
	    post {
		    always {
			    dependencyCheckPublisher pattern: 'target/dependency-check-report.xml'
		    }
	    } 
    }
    stage('Docker build and push') {
            steps {
                sh 'sudo docker build -t prakhar0012/numeric-app:${GIT_COMMIT} . '
                sh 'echo $DOCKER_CRED_PSW | docker login -u $DOCKER_CRED_USR --password-stdin'
                sh 'docker push prakhar0012/numeric-app:${GIT_COMMIT}'
      }
    }
    stage ('Vulnerability Scan - k8s') {
	    steps {
		    parallel(
		           "OPA Scan": {
		             sh 'docker run --rm -v $(pwd):/project openpolicyagent/conftest test --policy opa-k8s-security.rego k8s_deployment_service.yaml'
		           },
		           "Kubesec Scan": {
		             sh "bash kubesec-scan.sh"
           		   }
		   )
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
    stage('Integration Tests - DEV') {
      steps {
         script {
           try {
             withKubeConfig([credentialsId: 'kubeconfig']) {
               sh "bash integration-test.sh"
             }
           } catch (e) {
             withKubeConfig([credentialsId: 'kubeconfig']) {
               sh "kubectl -n default rollout undo deploy ${deploymentName}"
             }
             throw e
           }
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
