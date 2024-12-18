pipeline {
    agent any
    tools{
        jdk 'jdk17'   #jdk-tool type,jdk17-name configured in global tool
        maven 'maven3' #maven-tool type,maven3-name configured in global tool
    }
    environment{
        SCANNER_HOME= tool 'sonar-scanner'  #name in tool configuration
    }

    stages {
        stage('git-checkout') {
            steps {
                git 'https://github.com/jaiswaladi246/secretsanta-generator.git'
            }
        }

        stage('Code-Compile') {    # to find out any syntax based errors in our code
            steps {
               sh "mvn clean compile"
            }
        }
        
        stage('Unit Tests') {
            steps {
               sh "mvn test -DskipTests=true"  # when u want to build the project skip the time-consuming or unnecessary tests during that process.
            }
        }
	 stage('File System Scan') {
            steps {
                sh "trivy fs --format table -o trivy-fs-report.html ."
            }
        }   
        
	stage('OWASP Dependency Check') {
            steps {
               dependencyCheck additionalArguments: ' --scan ./ ', odcInstallation: 'DC'
                    dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }


        stage('Sonar Analysis') {                       #if u want 2 acccess any server we need servel url and username,password that we configured in system
            steps {
               withSonarQubeEnv('sonar'){               #name in system
                   sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Santa \
                   -Dsonar.java.binaries=. \
                   -Dsonar.projectKey=Santa '''
               }
            }
        }
	stage('Quality Gate') {
            steps {
                script {
                  waitForQualityGate abortPipeline: false, credentialsId: 'sonar-token'  #sonar-token-contains secret text in credentials
                }
            }
        }    

		 
        stage('Code-Build') {
            steps {
               sh "mvn package -DskipTests=true"  #Tests can take a long time to run Skipping them helps you quickly build the application
            }
        }

         stage('Docker Build') {
            steps {
               script{
                   withDockerRegistry(credentialsId: 'docker-cred') {
                    sh "docker build -t  santa123 . "
                 }
               }
            }
        }

        stage('Docker Push') {
            steps {
               script{
                   withDockerRegistry(credentialsId: 'docker-cred') {
                    sh "docker tag santa123 adijaiswal/santa123:latest"
                    sh "docker push adijaiswal/santa123:latest"
                 }
               }
            }
        }
        
        	 
        stage('Docker Image Scan') {
            steps {
               sh "trivy image adijaiswal/santa123:latest "
            }
        }}
        
         post {
            always {
                emailext (
                    subject: "Pipeline Status: ${BUILD_NUMBER}",
                    body: '''<html>
                                <body>
                                    <p>Build Status: ${BUILD_STATUS}</p>
                                    <p>Build Number: ${BUILD_NUMBER}</p>
                                    <p>Check the <a href="${BUILD_URL}">console output</a>.</p>
                                </body>
                            </html>''',
                    to: 'jaiswaladi246@gmail.com',
                    from: 'jenkins@example.com',
                    replyTo: 'jenkins@example.com',
                    mimeType: 'text/html'
                )
            }
        }
		
		

    
}
