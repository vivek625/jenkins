pipeline {
    agent any
    
    tools {
        jdk 'jdk11'
        maven 'maven3'
    }

    environment {
        SCANNER_HOME= tool 'sonar-scanner'
    }
    
    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'main', changelog: false, poll: false, url: 'https://github.com/Reliable-Royalty-29/SpringBoot-WebApplication.git'
            }
        }

        stage('Run Test Cases') {
            steps {
                sh "mvn test"
            }
        }
        
        stage('Build Artifact') {
            steps {
                sh "mvn clean package"
            }
        }
        
        stage('Sonarqube Static Code Analysis') {
            steps {
                script {
                    withSonarQubeEnv('sonar-server') {
                        sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Netflix-Website \
                        -Dsonar.java.binaries=. \
                        -Dsonar.projectKey=Netflix-Website '''
                    }
                }
            }
        }
       stage('Project Dependency Check Stage') {
            steps {
                dependencyCheck additionalArguments: '--scan ./', odcInstallation: 'Dependency-Check'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        
    }
}
