pipeline {
  agent any

  stages {
      stage('Build Artifact') {
            steps {
              sh "mvn clean package -DskipTests=true"
              archive 'target/*.jar' //so that they can be downloaded later
            }
        }   
       //--------------------------
    stage('UNIT test & jacoco ') {
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
    //---------------------------
    stage('Mutation Tests - PIT') {
      steps {
        catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
          sh "mvn org.pitest:pitest-maven:mutationCoverage"
        }
      }
        post { 
         always { 
           pitmutation mutationStatsFile: '**/target/pit-reports/**/mutations.xml'
         }
       }
      
    }
       //---------------------------
     stage('Sonarqube Analyse') {
      steps {
        catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
          withCredentials([string(credentialsId: 'token_sonar', variable: 'TOKEN')]){
          sh "mvn sonar:sonar \
  -Dsonar.projectKey=sohel \
  -Dsonar.host.url=http://devsecopstp1.eastus.cloudapp.azure.com:9999 \
  -Dsonar.login=${TOKEN}"
          }
        }
      }
     }
	  
//---------------------------
    stage('Vulnerability Scan owasp - dependency-check') {
   steps {
	    catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
     		sh "mvn dependency-check:check"
	    }
		}
}
     
//--------------------------


//--------------------------
    stage('Docker Build and Push') {

      steps {

        withCredentials([string(credentialsId: 'DOCKER_HUB_SOHEL', variable: 'DOCKER_HUB_PASSWORD')]) {

          sh 'sudo docker login -u sohel19 -p $DOCKER_HUB_PASSWORD'

          sh 'printenv'

          sh 'sudo docker build -t sohel19/devops-app:""$GIT_COMMIT"" .'

          sh 'sudo docker push sohel19/devops-app:""$GIT_COMMIT""'

        }
 
      }

    }
//--------------------------

   stage('scan trivy') {
       steps {
	              echo "sed -i 's#token_github#g' trivy-image-scan.sh"
                 //sh "sudo bash trivy-image-scan.sh"
	       
		
       }
     }


 //--------------------------
stage('Deployment Kubernetes  ') {
      steps {
        withKubeConfig([credentialsId: 'kubeconfigsohel']) {
              sh "sed -i 's#replace#sohel19/devops-app:${GIT_COMMIT}#g' k8s_deployment_service.yaml"
              sh 'kubectl apply -f k8s_deployment_service.yaml'
        }
      }
    }
//--------------------------


	  
    }
}
