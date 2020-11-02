def JENKINS_URL = "http://35.232.130.43:8080/"
properties([pipelineTriggers([githubPush()])])
pipeline {
	
	 
  agent any
  environment {
        JENKINS_URL = "${env.JENKINS_URL}"

    }
	
 // triggers {
        //pollSCM 'H/2 * * * *'
   // }
  
  tools {
    maven "maven latest"
  }
	
	
  
  stages{
    
    
    stage('Build') {
      steps {
              sh 'mvn -Dmaven.test.failure.ignore=true install' 
              //sh "mvn clean compile"
              slackSend channel: "#alerts", message: "Build Started:" + JENKINS_URL + "job/" + env.JOB_NAME+"/"+ env.BUILD_NUMBER
	      
        
        echo 'Build Done'  
      }
    }
    
    
    
    stage('SonarQube Analysis') {
	    		 environment {
       	 		scannerHome = tool 'sonarqubescanner'
    					}		
	    steps{
		    echo 'commented Sonarqube analysis'
        	//withSonarQubeEnv(credentialsId: 'sonar', installationName:'sonarqube') { 
		    //withSonarQubeEnv('sonarqube') {
       		//sh 'mvn clean package sonar:sonar -Dsonar.host.url=http://35.232.130.43:9000// -Dsonar.sources=. -Dsonar.tests=. -Dsonar.test.inclusions=**/test/java/servlet/createpage_junit.java -Dsonar.exclusions=**/test/java/servlet/createpage_junit.java -Dsonar.login=admin -Dsonar.password=admin'
		slackSend channel: "#alerts", message: "SonarQube Analysis Done successfully"
        //}
	// timeout(time: 10, unit: 'MINUTES') {
         //   waitForQualityGate abortPipeline: true
       // }
      
  }
    }
    
    
	
	
	stage('Deploy to Test') {
		steps{
			deploy adapters: [tomcat8(credentialsId: 'tomcat', path: '', url: 'http://34.123.69.37:8080/')], contextPath: '/QAWebapp', war: '**/*.war'
		
			   slackSend channel: "#alerts", message: "Deployed to Test server"
			
		}
    }
	 
	  
	  
	  
	  
	  
	  
	  
	  
	  stage ('Server config') {
            steps { 
              rtServer (
    id: 'artifactory',
    url: 'https://arunsahu2222.jfrog.io/artifactory',
    // If you're using username and password:
    username: 'deploy1',
    password: '10@Storage'

            )
            }
   }
//}
      stage ('Upload file') {
            steps {
                rtUpload (
                    serverId: 'artifactory',
                    spec: """{
                            "files": [
                                    {
                                        "pattern": "*.war",
                                        "target": "libs-release-local"
                                    }
                                ]
                            }"""
                )
               
              rtPublishBuildInfo (
    serverId: 'artifactory')
   
            }
        }

	  
	  
	  
	  
	  
	  
	  
	  
	  
	  
	  
	  
	  
	   stage('UI Test') {
		   steps{
				publishHTML([escapeUnderscores:true,allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: '\\functionaltest\\target\\surefire-reports', reportFiles: 'index.html', reportName: 'UI_TEST_Report', reportTitles: 'HTML Report'])
			   slackSend channel: "#alerts", message: "UI Test report published"
		   }
    			}
	  
	  
	   stage('Performance Test') {
		 steps{
			echo 'BlazeMeterTest' 
			blazeMeterTest credentialsId: 'blazemeter', testId: '8491749.taurus', workspaceId: '648314'
			 slackSend channel: "#alerts", message: "Performance test report published"
		   }
    }
	  
	  
	  stage('Deploy to Prod') {
		  steps{
	      deploy adapters: [tomcat8(credentialsId: 'tomcat', path: '', url: 'http://35.193.167.170:8080/')], contextPath: '/ProdWebapp', war: '**/*.war'
			  slackSend channel: "#alerts", message: "Deployed to prod"
		  }
         }
	  
	  
	  stage('Sanity Test') {
		  steps{
	publishHTML([escapeUnderscores:true,allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: '\\Acceptancetest\\target\\surefire-reports', reportFiles: 'index.html', reportName: 'Sanity Test Report', reportTitles: 'HTML Report'])
			  slackSend channel: "#alerts", message: "Sanity Test report published"
		  }
    }
	
	
	

    stage('Send Notification') {
      steps {
        	 slackSend channel: '#devops-learning', message: "Pipeline Completed ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${JENKINS_URL + "job/" + env.JOB_NAME+"/"+ env.BUILD_NUMBER}|Open>)"
	      echo 'notification sent to channel: devops-learning' 

      }
    }

  }
}
