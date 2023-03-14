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
        }

      stage('Mutation test - PIT') {
            steps {
              sh "mvn org.pitest:pitest-maven:mutationCoverage"
            }
        }

      stage('Sonarqube - ASAT') {
        steps {
	  withSonarQubeEnv('SonarQube'){
		  sh "mvn clean verify sonar:sonar -Dsonar.projectKey=numeric-application -Dsonar.host.url=http://asir-devsecops-practices.eastus.cloudapp.azure.com:9000 -Dsonar.login=sqp_16220815302119c51022666b8dae81f08cf75b3c"
	  }
          timeout(time: 2, unit: 'MINUTES') {
	    script {
	      waitForQualityGate abortPipeline: true
	    }
	  }
        }
      }


      stage('Vulnerability Scan - Docker') {
        steps {
          parallel(
            "Dependency Scan": {
              sh "mvn dependency-check:check"
            },
            "Trivy Scan": {
              sh "bash trivy-docker-image-scan.sh"
            }
          )
        }
      }

      stage('Docker build and push'){
          steps {
              withDockerRegistry([credentialsId: "dockerhub", url: ""]) {
                  sh 'printenv'
                  sh 'chown -R jenkins. /var/lib/jenkins/workspace/devsecops-numeric-appllication/trivy'
                  sh 'docker build -t robertoasir/numeric-app:""$GIT_COMMIT"" .'
                  sh 'docker push robertoasir/numeric-app:""$GIT_COMMIT""'
                }
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

  post {
    always {
      dependencyCheckPublisher pattern: 'target/dependency-check-report.xml'
      junit 'target/surefire-reports/*.xml'
      jacoco execPattern: 'target/jacoco.exec'
  		pitmutation mutationStatsFile: '**/target/pit-reports/**/mutations.xml'
    }
  }
}
