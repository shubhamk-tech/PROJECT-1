pipeline {
    agent any
    
    tools {
        jdk 'jdk17'
        maven 'maven3'
    }
    
    environment {
        SCANNER_HOME= tool 'sonar-scanner'
    }
    
    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'main', credentialsId: 'github', url: 'https://github.com/shubhamk-tech/PROJECT-1.git'
            }
        }
        
        stage('Compile Code') {
            steps {
                sh "mvn compile"
            }
        }
        
        stage('Test Cases') {
            steps {
                sh "mvn test -DskipTests=true"
            }
        }
        
        stage('Trivy Scan') {
            steps {
                sh "trivy fs --format table -o trivy-fs-report.html ."
            }
        }
        
        stage('Sonarqube Analysis') {
            steps {
                withSonarQubeEnv('sonar') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectKey=Mission -Dsonar.projectName=Mission \
                            -Dsonar.java.binaries=. '''
                }
            }
        }
        
        stage('Build the code') {
            steps {
                sh "mvn package -DskipTests=true"
            }
        }
        
        stage('Build & Tag Docker Images') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                        sh "docker build -t shubhz1452/mission:1 ."
                    }    
                }
            }
        }
        
        stage('Trivy Scan Docker Image') {
            steps {
                sh "trivy fs . > trivyfs.txt"
            }
        }
        
        stage('Publish Docker Images') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                        sh "docker push shubhz1452/mission:1"
                    }
                }
            }
        }
        
        stage('Deploy to K8S') {
            steps {
                withKubeConfig(caCertificate: '', clusterName: 'my-eks-prod', contextName: '', credentialsId: 'k8s-token', namespace: 'webapps', restrictKubeConfigAccess: false, serverUrl: 'https://D27BB86303FA9A33C78AD09744F6DB94.gr7.us-west-2.eks.amazonaws.com') {
                    sh "kubectl apply -f k8s/ds.yaml -n webapps"
                    sleep 60
                }
            }
        }
        
        stage('Verify Deployment') {
            steps {
                withKubeConfig(caCertificate: '', clusterName: 'my-eks-prod', contextName: '', credentialsId: 'k8s-token', namespace: 'webapps', restrictKubeConfigAccess: false, serverUrl: 'https://D27BB86303FA9A33C78AD09744F6DB94.gr7.us-west-2.eks.amazonaws.com') {
                    sh "kubectl get pods -n webapps"
                    sh "kubectl get svc -n webapps"
                }
            }
        }
    }
}

