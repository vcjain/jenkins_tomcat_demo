pipeline {
    agent any
    tools {
        maven 'maven3'
    }
    stages{
        
        stage('checkout'){
            steps{
                checkout scmGit(branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[url: 'https://github.com/vcjain/jenkins_tomcat_demo.git']])
            }    
        }
        stage('Build'){
            steps{
                echo 'Building Maven project'
                sh 'mvn clean install'
            }
        }
        stage('Deploy on Tomcat'){
            steps{
                echo 'Deploying on Tomcat '
                sshagent(['tomcat-key']) {
                    sh 'scp -v -o StrictHostKeyChecking=no target/demowar-0.0.1-SNAPSHOT.war ubuntu@$TOMCAT_IP:/opt/tomcat/webapps'
                }
            }
        }
        
    }
        

}
