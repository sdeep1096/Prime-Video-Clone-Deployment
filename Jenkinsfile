pipeline{
    agent any
    tools{
        jdk 'jdk17'
        nodejs 'node23'
    }
    environment{
        SCANNER_HOME=tool 'sonar-scanner'
    }
    stages{
        stage('Clean Workspace') {
            steps{
                clwanWs()
            }
        }
        stage('Git Checkout'){
            steps{
                checkout scmGit(branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[url: 'https://github.com/sdeep1096/Prime-Video-Clone']])
            }
        }
        stage('Sonarqube Analysis'){
            steps{
                withSonarQubeEnv('sonar-server'){
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=amazon-prime \
                    -Dsonar.projectKey=amazon-prime '''
                }
            }
        }
        stage('quality gate'){
            steps{
               script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'sonar-token' 
                } 
            }
        }
        stage('Install NPM dependencies'){
            steps{
                sh "npm install"
            }
        }
        stage('OWASP FS SCAN') {
            steps {
                dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit', odcInstallation: 'DP-Check'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        stage('Trivy File Scan'){
            steps{
                sh "trivy fs . > trivy.txt"
            }
        }
        stage('Docker Build Image'){
            steps{
                sh "docker build -t briarheart1096/amazon-prime:latest ."
            }
        }
        stage('Push Image to Docker Hub'){
            steps{
                sh "docker push briarheart1096/amazon-prime:latest"
            }
        }
        stage('Docker Scout Image') {
            steps {
                script{
                   withDockerRegistry(credentialsId: 'docker', toolName: 'docker'){
                       sh 'docker-scout quickview briarheart1096/amazon-prime:latest'
                       sh 'docker-scout cves briarheart1096/amazon-prime:latest'
                       sh 'docker-scout recommendations briarheart1096/amazon-prime:latest'
                   }
                }
            }
        }
        stage('Deploy Container'){
            steps{
                sh "docker run -d --name amazon-prime -p 3000:3000 briarheart1096/amazon-prime:latest"
            }
        }
    }
    post {
    always {
        emailext attachLog: true,
            subject: "'${currentBuild.result}'",
            body: """
                <html>
                <body>
                    <div style="background-color: #FFA07A; padding: 10px; margin-bottom: 10px;">
                        <p style="color: white; font-weight: bold;">Project: ${env.JOB_NAME}</p>
                    </div>
                    <div style="background-color: #90EE90; padding: 10px; margin-bottom: 10px;">
                        <p style="color: white; font-weight: bold;">Build Number: ${env.BUILD_NUMBER}</p>
                    </div>
                    <div style="background-color: #87CEEB; padding: 10px; margin-bottom: 10px;">
                        <p style="color: white; font-weight: bold;">URL: ${env.BUILD_URL}</p>
                    </div>
                </body>
                </html>
            """,
            to: 'provide_your_Email_id_here',
            mimeType: 'text/html',
            attachmentsPattern: 'trivy.txt'
        }
    }
}
