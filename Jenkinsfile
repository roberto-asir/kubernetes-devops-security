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
	  sh "mvn clean verify sonar:sonar -Dsonar.projectKey=numeric-application -Dsonar.host.url=http://asir-devsecops-practices.eastus.cloudapp.azure.com:9000 -Dsonar.login=sqp_16220815302119c51022666b8dae81f08cf75b3c"
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
