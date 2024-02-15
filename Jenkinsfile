pipeline {
    agent any
    
    tools {
        jdk 'jdk17'
    }
    
      environment {
        SCANNER_HOME=tool 'sonar-scanner'
    }

    stages {
        stage('Git Checkout') {
            steps {
                git 'https://github.com/shivu24-abylle/DotNet-DEMO.git'
            }
        }
        
        stage('OWASP Dependency') {
            steps {
                dependencyCheck additionalArguments: '--scan ./ ', odcInstallation: 'DC'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        stage('Trivy FS') {
            steps {
               sh "trivy fs ."
            }
        }
        stage(" Sonarqube Analysis "){
            steps{
                 withSonarQubeEnv('sonar') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Dotnet-demo \
                    -Dsonar.projectKey=Dotnet-demo '''
                    
                 }
            }
        }
        stage('Docker build and tag') {
            steps {
               script{
                   withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                    sh "make image"
                    }
               }
            }
        }
        stage("Trivy Image Scan "){
            steps{
                //sh "trivy image sushantkapare1717/dotnet-demoapp "
                sh "trivy image shivakumar24041993/dotnet-demoapp "
            }
        } 
         
        stage('Build and Push') {
            steps {
                script {
                    // Build your Docker image here

                    // Log in to Docker Hub or your private registry
                    docker.withRegistry('https://registry.hub.docker.com', 'docker-cred') {
                        // Push the Docker image
                        docker.image('shivakumar24041993/dotnet-demoapp').push()
                        
                    }
                }
            }
        }
        stage('Deloy to container ') {
            steps {
               script{
                   withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                    sh "docker run -d -p 5000:5000  shivakumar24041993/dotnet-demoapp "
                    }
               }
            }
        }
        
    }
}
