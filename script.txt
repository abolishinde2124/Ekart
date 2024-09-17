pipeline {
    agent any
    tools {
        jdk 'jdk17'
        maven 'maven3'
    }
    environment {
        SCANNER_HOME = tool 'sonar-scanner'
    }
    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'main', changelog: false, credentialsId: '0a68806d-b9c3-41e9-adac-4a91a1929c36', poll: false, url: 'https://github.com/abolishinde2124/Ekart.git'
            }
        }
        
        stage('COMPILE') {
            steps {
                sh 'mvn clean package -DskipTests=true'  // Use package instead of compile to generate the JAR
            }
        }
    
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonar-server') {
                    sh '''$SCANNER_HOME/bin/sonar-scanner \
                        -Dsonar.projectName=Ekart \
                        -Dsonar.projectKey=Ekart \
                        -Dsonar.java.binaries=target/classes \
                        -Dsonar.login=squ_a0495e974d95a5aae3ee1f1e52a65ff875bb38df '''
                }
            }
        }
        
        stage('Docker build & push') {
            steps {
                withDockerRegistry(credentialsId: 'b143bb95-2bfc-4516-9680-15a7f70be34b', url: 'https://index.docker.io/v1/') {
                    sh "docker build -t abolishinde21/ekart:latest -f docker/Dockerfile ."
                    sh "docker push abolishinde21/ekart:latest"
                }
            }
        }
        stage('Depoly') {
            steps {
                withDockerRegistry(credentialsId: 'b143bb95-2bfc-4516-9680-15a7f70be34b', url: 'https://index.docker.io/v1/') {
                    sh "docker run -d --name shop-shop -p 8070:8070 abolishinde21/ekart:latest"
                }
            }
        }
    }
}
