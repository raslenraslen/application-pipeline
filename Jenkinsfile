pipeline {
    agent any
    
    tools {
        maven 'Maven3'
        jdk 'JDK'
    }
      environment {
        
        SONAR_HOME = tool 'sonar-scanner'
    }
    

    stages { 
        
         stage('Git checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/raslenraslen/application-pipeline.git'
            }
        }
        stage('Maven Compile') {
            steps {
               sh "mvn compile"
            }
        }
        
        stage(' Maven Test') {
            steps {
                sh "mvn test"
            }
        }
        
        stage('Build') {
            steps {
                sh "mvn package"
            }
        }
          stage('Trivy FS scan') {
            steps {
            sh " trivy fs --format table -o fs.html ."
                archiveArtifacts artifacts: 'fs.html', fingerprint: true
            }
        }
         stage('Sonarqube Analysis') {
            steps {
            withSonarQubeEnv( 'sonar-server') {
                sh ''' $SONAR_HOME/bin/sonar-scanner -Dsonar.projectName=Raslen -Dsonar.projectKey=Raslen \
                        -Dsonar.java.binaries=target '''
}
               
            }
        }
        stage('Publish to Nexus') {
            steps {
                configFileProvider([configFile(fileId: 'fb0f3413-0523-47bb-b0cc-fe2e4c17124b', variable: 'mavensettings')]) {
                    
                    sh 'mvn -s "$mavensettings" clean deploy'
                }
            }
        }
         stage('Build & Tag Docker Imagee') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                        sh "docker build -t raslenmissaoui061/raslenshackk:latest ."
                    }
                }
            }
        }
          stage('Push Docker Image') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                        sh "docker push raslenmissaoui061/raslenshackk:latest "
                    }
                }
            }
        }
         stage(' Docker Image Scan') {
            steps {
               
                sh '    trivy image --format table -o trivy-fs-report.html raslenmissaoui061/raslenshackk:latest '
            
           
            }
        }
        stage('Deploy To Kubernetes') {
            steps {
                withKubeCredentials(kubectlCredentials: [[caCertificate: '', clusterName: 'kubernetes', contextName: '', credentialsId: 'k8s-token2', namespace: 'pipeline', serverUrl: 'https://192.168.100.64:6443']])  {
                    sh "kubectl apply -f deployment-service.yaml --force"
                }
            }
        }
         
        
        stage('Verify Deployment') {
            steps {
                withKubeCredentials(kubectlCredentials: [[caCertificate: '', clusterName: 'kubernetes', contextName: '', credentialsId: 'k8s-token2', namespace: 'pipeline', serverUrl: 'https://192.168.100.64:6443']])  {
                    sh "kubectl get svc -n pipeline"
                }
            }
        }
    }
}
