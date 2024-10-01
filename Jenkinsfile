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
    //---------------------------


	  
    }
}
