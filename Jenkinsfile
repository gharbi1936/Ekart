pipeline {
    agent any
    // Declare tools for JDK 11 and Maven
    tools {
        jdk 'jdk11'
        maven 'mvn3'
        dockerTool 'docker'
    }
    
    environment {
        // SonarQube Scanner installations// SonarQube Scanner tool name = "sonar-scaner" 
        SONAR_SCANNER_HOME = tool 'sonar-scanner'
        IMAGE = gharbi1936/shopping-card:v0.0.1
        
    }

    stages {
        stage('checkout') {
            steps {
                git branch: 'main', credentialsId: 'git_token9', url: 'https://github.com/gharbi1936/Ekart.git'
            }
        }
        stage('compile') {
            steps {
                sh 'mvn clean compile -DskipTests=true'
            }
        }
        stage('owasp scan') {
            steps {
                dependencyCheck additionalArguments: '--scan ./ ', odcInstallation: 'dp'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
                
            }
        }
        stage('sonar scan') {
            steps {
                withSonarQubeEnv('sonar-server') {
                    sh "$SONAR_SCANNER_HOME/bin/sonar-scanner \
                        -Dsonar.projectName=Shopping-Cart \
                        -Dsonar.java.binaries=. \
                        -Dsonar.projectKey=Shopping-Cart"
                }
            }
        }
        stage('build') {
            steps {
                sh 'mvn clean package -DskipTests=true'
            }
        }
        stage('image') {
            steps {
                // sh '''
                // echo "User:"
                // whoami
                // echo "User ID:"
                // id -u
                // echo "Current working directory:"
                // pwd
                // echo "Docker version:"
                // docker version
                // echo "User group permissions:"
                // groups
                // '''
                sh "docker build -t $IMAGE -f docker/Dockerfile ."
            }
        }
        stage('push') {
            steps {
                script{
                    // Authenticate with Docker registry
                    withDockerRegistry(credentialsId: 'docker', toolName: 'docker' ){ 
                    // Push Docker image to the registry
                    sh "docker push $IMAGE"
                    }
                }
            }
        }
        stage('trivy') {
            steps {
                sh 'trivy image --no-progress --severity MEDIUM,HIGH,CRITICAL --exit-code 0 gharbi1936/shopping-card:v0.0.1'
            }
        }
        stage('deploy') {
            steps {
                script{
                    // Authenticate with Docker registry
                    withDockerRegistry(credentialsId: 'docker', toolName: 'docker' ){ 
                    // Push Docker image to the registry
                    sh "docker run -d -p 8070:8070 --name container-branch-main $IMAGE "
                    }
                }
            }
        }
    }
}