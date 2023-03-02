pipeline {
  agent any

  stages {
      stage('Build Artifact') {
            steps {
              sh "mvn clean package -DskipTests=true"
              archive 'target/*.jar' 
            }
        }   

      stage('Unit test') {
            steps {
              sh "mvn test"
            }
	    post {
	      always {
		junit 'target/surefire-reports/*.xml'
		jacoco execPattern: 'target/jacoco.exec'
	      }
	    }
        }

      stage('Mutation test - PIT') {
            steps {
              sh "mvn org.pitest:pitest-maven:mutationCoverage"
            }
	    post {
	      always {
		pitmutation mutationStatsFile: '**/target/pit-reports/**/mutations.xml'
	      }
	    }
        }

      stage('Docker build and push'){
          steps {
              withDockerRegistry([credentialsId: "dockerhub", url: ""]) {
                  sh 'printenv'
                  sh 'docker build -t robertoasir/numeric-app:""$GIT_COMMIT"" .'
                  sh 'docker push robertoasir/numeric-app:""$GIT_COMMIT""'
                }
            }
	}

      stage('Sonarqube - ASAT') {
        steps {
	  sh "mvn clean verify sonar:sonar -Dsonar.projectKey=devsecops -Dsonar.host.url=http://asir-devsecops-practices.eastus.cloudapp.azure.com:9000 -Dsonar.login=sqp_3cd33e21295aa3c96f8f3333b2bfff7d53b955e9"
        }
      }
  
      stage('Kubernetes Deployment - DEV'){
          steps {
              withKubeConfig([credentialsId: 'kubeconfig']) {
		  sh "sed -i 's#replace#robertoasir/numeric-app:${GIT_COMMIT}#g' k8s_deployment_service.yaml"
                  sh "kubectl apply -f k8s_deployment_service.yaml"
                }
            }
        } 
    }
}
