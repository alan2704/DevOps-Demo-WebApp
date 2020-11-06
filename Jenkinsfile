def JENKINS_URL = "http://3.17.141.33:8080/"

pipeline {
  agent any
	
  tools {
    maven "maven"
  }
  
  stages{
	  

	  
	  stage('Checkout') {
		  steps	  {
        		//git url: 'https://github.com/tipurmadan/DevOps-Demo-WebApp.git'
			checkout([$class: 'GitSCM', branches: [[name: '*/master']], userRemoteConfigs: [[url: 'https://github.com/alan2704/DevOps-Demo-WebApp.git']]])
			  	
			        
				
			      
		  }
    }
	  
	  
    stage('Build') {
      steps {
              //sh 'mvn -Dmaven.test.failure.ignore=true install' 
             // sh "mvn clean install"
              slackSend channel: "#alerts", message: "Build Started:" + JENKINS_URL + "job/" + env.JOB_NAME+"/"+ env.BUILD_NUMBER
	      jiraSendBuildInfo branch: 'master', site: 'squad-3-devops.atlassian.net'
        
        echo 'Build Done' 
      }
	
   }
    
    
    
    //stage('Static code Analysis') {
//	    		 environment {
   //    	 			scannerHome = tool 'sonarqubescanner'
    //				}		
	//    steps{
	//	   withSonarQubeEnv('sonarqube') {
       //		sh 'mvn clean package sonar:sonar -Dsonar.sources=. -Dsonar.tests=. -Dsonar.test.inclusions=**/test/java/servlet/createpage_junit.java -Dsonar.exclusions=**/test/java/servlet/createpage_junit.java -Dsonar.login=admin -Dsonar.password=admin'
	//	slackSend channel: "#alerts", message: "SonarQube Analysis Done successfully"
      //  }
	
      
  //}
 //   }
    
    
	
	
	stage('Deploy to Test') {
		steps{
			deploy adapters: [tomcat8(credentialsId: 'tomcat', path: '', url: 'http://3.138.191.211:8080/')], contextPath: '/QAWebapp', war: '**/*.war'
		
			   slackSend channel: "#alerts", message: "Deployed to Test server"
				 jiraAddComment idOrKey: "${'SQUAD3-2'}", site: 'jirasite' , comment: "${currentBuild.getCurrentResult()}" + ' Code deployed to Test  ' + 'SQUAD3-2'
        			jiraTransitionIssue idOrKey: "${'SQUAD3-2'}", input: [transition: [id: '31']] , site: 'jirasite'
			      jiraSendBuildInfo branch: "${'SQUAD3-2'}", site: 'squad-3-devops.atlassian.net'
				//jiraSendDeploymentInfo environmentId: 'envid', environmentName: 'test-env-name', environmentType: 'testing',  serviceIds: ['http://35.225.186.14:8080/QAWebapp'],  site: 'squad-3-devops.atlassian.net', state: 'successful'			      
				jiraSendDeploymentInfo environmentId: 'envid', environmentName: 'test-env-name', environmentType: 'testing',   site: 'squad-3-devops.atlassian.net', state: 'successful'			      

		}
    }
	 
	  
	  
	
	  
	  
	  
	  
	  
	  
//}
      stage ('Deploy Artifacts') {
            steps {
		    
	    
                rtUpload (
                    serverId: 'artifactory',
                    spec: """{
                            "files": [
                                    {
                                        "pattern": "**/*.war",
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
			    	sh 'mvn test -f functionaltest/pom.xml'
				publishHTML([escapeUnderscores:true,allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: '\\functionaltest\\target\\surefire-reports', reportFiles: 'index.html', reportName: 'UI_TEST_Report', reportTitles: 'HTML Report'])
			   slackSend channel: "#alerts", message: "UI Test report published"
		   }
    			}
	  
	  
	   //stage('Performance Test') {
	//	 steps{
	//		echo 'BlazeMeterTest' 
	//		blazeMeterTest credentialsId: 'blazemeter', testId: '8487271.taurus', workspaceId: '646655'
	//		 slackSend channel: "#alerts", message: "Performance test report published"
	//	   }
 //  }
	  
	  
	  stage('Deploy to Prod') {
		  steps{
	      deploy adapters: [tomcat8(credentialsId: 'tomcat', path: '', url: 'http://3.139.103.161:8080/')], contextPath: '/ProdWebapp', war: '**/*.war'
			  slackSend channel: "#alerts", message: "Deployed to prod"
				 jiraAddComment idOrKey: "${'SQUAD3-2'}", site: 'jirasite' , comment: "${currentBuild.getCurrentResult()}" + ' Code deployed to Test  ' + 'SQUAD3-2'
        			jiraTransitionIssue idOrKey: "${'SQUAD3-2'}", input: [transition: [id: '31']] , site: 'jirasite'
			      jiraSendBuildInfo branch: "${'SQUAD3-2'}", site: 'squad-3-devops.atlassian.net'
				jiraSendDeploymentInfo environmentId: 'envid', environmentName: 'test-env-name', environmentType: 'testing',  site: 'squad-3-devops.atlassian.net', state: 'successful'			      
		  }
		  
         }
	  
	  
	  stage('Sanity Test') {
		  steps{
			   sh 'mvn test -f Acceptancetest/pom.xml'
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
	post { 
		failure {
			slackSend channel: '#devops-learning', color:'RED', message: "Pipeline FAILURE ${env.JOB_NAME} #${env.BUILD_NUMBER}"
		}
		success {
			slackSend channel: '#devops-learning', color:'good', message: "Pipeline Completed ${currentBuild.fullDisplayName} successfully"
		}
	}
	
  stage('Tools Init') {
            steps {
                script {
                    echo "PATH = ${PATH}"
                    echo "M2_HOME = ${M2_HOME}"
               def tfHome = tool name: 'Ansible'
                env.PATH = "${tfHome}:${env.PATH}"
                 sh 'ansible --version'
                   
            }
            }
        }
	
 stage('Ansible Deploy') {
            
            steps {
                
           sh "ansible-playbook main.yml -i inventories/dev/hosts -- user jenkins --key-file ~/.ssh/id_rsa"
}
}	
		
}
