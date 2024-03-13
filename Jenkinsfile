pipeline{
    agent any
    tools{
        jdk 'jdk17'
        maven 'mvn3'
    }
    environment{
        SCANNER_HOME=tool 'sonar-scanner'
    }
    stages{
        stage('git checkout'){
            steps{
                git branch: 'main', url: 'https://github.com/mukeshr-29/project-35-java_application-boardgame_listing.git'
            }
        }
        stage('code compile'){
            steps{
                sh 'mvn compile'
            }
        }
        stage('unit testing'){
            steps{
                sh 'mvn test'
            }
        }
        stage('file system scan'){
            steps{
                sh 'trivy fs --format table -o trivyfs-report.html .'
            }
        }
        stage('static code analysis using sonarqube'){
            steps{
                script{
                    withSonarQubeEnv(credentialsId: 'sonar-token', installationName: 'sonar-server'){
                        sh '''
                            $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Boardgamelist \
                                -Dsonar.projectKey=Boardgamelist \
                                -Dsonar.java.binaries=.
                        '''
                    }
                }
            }
        }
        stage('quality gate analysis'){
            steps{
                script{
                    waitForQualityGate abortPipeline: false, credentialsId: 'sonar-token'
                }
            }
        }
        stage('build application'){
            steps{
                sh 'mvn package'
            }
        }
        stage('publish built artifact to nexus'){
            steps{
                script{
                    withMaven(globalMavenSettingsConfig: 'global-maven', jdk: 'jdk17', maven: 'mvn3', mavenSettingsConfig: '', traceability: true){
                        sh 'mvn deploy'
                    }
                }
            }
        }
        stage('docker build and tag'){
            steps{
                script{
                    withDockerRegistry(credentialsId: 'dockerhub', toolName: 'docker'){
                        sh 'docker build -t mukeshr29/boardgamelist:latest .'
                    }
                }
            }
        }
        stage('docker img scan'){
            steps{
                sh 'trivy image --format table -o trivyimg-report.html mukeshr29/boardgamelist:latest'
            }
        }
        stage('docker img push'){
            steps{
                script{
                    withDockerRegistry(credentialsId: 'dockerhub', toolName: 'docker'){
                        sh 'docker push mukeshr29/boardgamelist:latest'
                    }
                }
            }
        }
        stage('deploy to k8s'){
            steps{
                script{
                    withKubeConfig(caCertificate: '', clusterName: 'kubernetes', contextName: '', credentialsId: 'k8s', namespace: 'webapps', restrictKubeConfigAccess: false, serverUrl: 'https://185.29.2.50:6443'){
                        sh 'kubectl apply -f deployment.yml -f service.yml' 
                    }
                }
            }
        }
        stage('verify deployment'){
            steps{
                script{
                    withKubeConfig(caCertificate: '', clusterName: 'kubernetes', contextName: '', credentialsId: 'k8s', namespace: 'webapps', restrictKubeConfigAccess: false, serverUrl: 'https://185.29.2.50:6443'){
                        sh 'kubectl get pods'
                        sh 'kubectl get svc'
                    }
                }
            }
        }
    }
    post{
    always {
        script {
            def jobName = env.JOB_NAME
            def buildNumber = env.BUILD_NUMBER
            def pipelineStatus = currentBuild.result ?: 'UNKNOWN'
            def bannerColor = pipelineStatus.toUpperCase() == 'SUCCESS' ? 'green' : 'red'
            def body = """
                <html>
                <body>
                <div style="border: 4px solid ${bannerColor}; padding: 
                10px;">
                <h2>${jobName} - Build ${buildNumber}</h2>
                <div style="background-color: ${bannerColor}; padding: 
                10px;">
                <h3 style="color: white;">Pipeline Status:
                ${pipelineStatus.toUpperCase()}</h3>
                </div>
                <p>Check the <a href="${BUILD_URL}">console 
                output</a>.</p>
                </div>
                </body>
                </html>
                """
            emailext (
                subject: "${jobName} - Build ${buildNumber} - ${pipelineStatus.toUpperCase()}",
                body: body,
                to: 'mukeshr2911@gmail.com',
                from: 'jenkins@example.com',
                replyTo: 'jenkins@example.com',
                mimeType: 'text/html',
                attachmentsPattern: 'trivyimg-report.html, trivyfs-report.html'
            )
            }
        }
    }
}