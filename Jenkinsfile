pipeline {
    agent any

    tools {
        jdk 'jdk11'
        maven 'maven3'
    }

    environment {
        SCANNER_HOME = tool 'sonar-scanner'
        PROJECT_NAME = 'Netflix-Website' // Define the project name here
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
                    withSonarQubeEnv('sonarqube1') {
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

        stage('Deploy Build Artifact to Nexus') {
            steps {
                configFileProvider([configFile(fileId: 'abdd1477-5e22-4da6-99d1-8817d595cc37', variable: 'mavensettings')]) {
                    sh 'mvn -s $mavensettings clean deploy'
                }
            }
        }

        stage('Build the Docker Image and Push to DockerHub') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'Docker') {
                        sh 'docker build -t netflix-website .'
                        sh 'docker tag netflix-website dheeman29/netflix-website:v2'
                        sh 'docker push dheeman29/netflix-website:v2'
                    }
                }
            }
        }

        stage('Docker Image Scan By Trivy') {
            steps {
                sh 'trivy image dheeman29/netflix-website:v2'
            }
        }
         stage('Deploy to Tomcat') {
            steps {
                sh 'sudo cp /var/lib/jenkins/workspace/"NETFLIX-CI"/target/spring-boot-web.jar /opt/apache-tomcat-9.0.65/webapps/'
            }
        }
       
    }

    // Send Slack notifications outside the stages block
    post {
        success {
            slackSend color: 'good', message: "Pipeline for Project: ${env.PROJECT_NAME} Succeeded! :tada:"
        }
        failure {
            slackSend color: 'danger', message: "Pipeline for Project: ${env.PROJECT_NAME} Failed! :fire:"
        }
    }
}
