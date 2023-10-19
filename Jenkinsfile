pipeline {
    agent any
    tools{
        jdk  'jdk17'
        maven  'maven3'
    }
    
    environment{
        SCANNER_HOME= tool 'sonar-scanner'
    }
    
    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'main', changelog: false, poll: false, url: 'https://github.com/RAJAMMA0808/secretsanta-generator.git'
            }
        }
        
        stage('COMPILE') {
            steps {
                sh "mvn clean compile -DskipTests=true"
            }
        }
        
        stage('OWASP Scan') {
            steps {
                dependencyCheck additionalArguments: '--scan ./ ', odcInstallation: 'DP-Check'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        
        stage('Sonarqube') {
            steps {
                withSonarQubeEnv('sonar-server') {
                   sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=santa \
                   -Dsonar.java.binaries=. \
                   -Dsonar.projectKey=santa '''
               }
            }
        }
        stage('Build') {
            steps {
                sh "mvn clean package -DskipTests=true"
            }
        }
        stage('Build Docker Image') {
            steps {
                script{
                    withDockerRegistry(credentialsId: 'docker',toolName: 'docker') {
				       sh "docker build -t santa:latest ."
                    }
                }
            }
        }
		stage('Tag & Push Docker Image') {
            steps {
                script{
                    withDockerRegistry(credentialsId: 'docker',toolName: 'docker') {
					    sh "docker tag santa raji0808/santa:latest"
						sh "docker push raji0808/santa:latest"
                    }
                }
            }
        }
        stage('Deploy Application') {
		    steps {
               script{
			    withDockerRegistry(credentialsId: 'docker',toolName: 'docker') {
					 sh "docker run -d -p 8081:8080 raji0808/santa:latest"
                    }
		        }
            }
         }
    }
}
