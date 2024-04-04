pipeline {
    agent { label 'jen-agents' }
    tools {
        jdk 'Java17'
        maven 'Maven3'
    }
    environment {
	    APP_NAME = "cicd-e2e-app"
            RELEASE = "1.0.0"
            DOCKER_USER = "shivabogem"
            DOCKER_PASS = 'dockerhb'
            IMAGE_NAME = "${DOCKER_USER}" + "/" + "${APP_NAME}"
            IMAGE_TAG = "${RELEASE}-${BUILD_NUMBER}"
	    JENKINS_API_TOKEN = credentials("JENKINS_API_TOKEN")
    }
   
    stages{
        stage("Cleanup Workspace"){
                steps {
                cleanWs()
                }
        }
        stage("Checkout from SCM"){
                steps {
                    git branch: 'main', credentialsId: 'github', url: 'https://github.com/Shivasai011/cicd-e2e-app.git'
		}
	}
	stage("Build Application"){
            steps {
                sh "mvn clean package"
            }
       }
	stage("Test Application"){
           steps {
                 sh "mvn test"
	   }
	}
	stage("SonarQube Analysis"){
           steps {
	           script {
		        withSonarQubeEnv(credentialsId: 'jenkins-sonar') { 
                        sh "mvn sonar:sonar"
		        }
		   }
	   }
	}
	stage("Quality Gate"){
           steps {
               script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'jenkins-sonar'
                }	
            }
        }
	stage("Build & Push Docker Image") {
            steps {
                script {
                    docker.withRegistry('',DOCKER_PASS) {
                        docker_image = docker.build "${IMAGE_NAME}"
                    }
                    docker.withRegistry('',DOCKER_PASS) {
                        docker_image.push("${IMAGE_TAG}")
                        docker_image.push('latest')
                    }
                }
            }
       }
	stage("Trigger CD Pipeline") {
            steps {
                script {
                    sh "curl -v -k --user clouduser:${JENKINS_API_TOKEN} -X POST -H 'cache-control: no-cache' -H 'content-type: application/x-www-form-urlencoded' --data 'IMAGE_TAG=${IMAGE_TAG}' 'ec2-3-87-87-128.compute-1.amazonaws.com:8080/job/continuous-delivery/buildWithParameters?token=gitops-token'"
                }
	    }
	}
    }
}
