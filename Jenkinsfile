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
                scripts{
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
                scripts{
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
                scripts{
                    withMaven(globalMavenSettingsConfig: 'global-maven', jdk: 'jdk17', maven: 'mvn3', mavenSettingsConfig: '', traceability: true){
                        sh 'mvn deploy'
                    }
                }
            }
        }
    }
}