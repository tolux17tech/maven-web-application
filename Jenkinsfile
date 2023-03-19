pipeline{
    agent any
    tools {
        maven "maven"
    }
    stages {
        
        stage('Git Checkout'){
            steps{
               git branch: 'master', url: 'https://github.com/tolux17tech/maven-web-application.git' 
            }
        }

        stage("Unit Testing"){
            steps{
                sh 'mvn test'
            }
        }

        stage("Integration Testing"){
            steps{
                sh 'mvn verify -DskipUnitTests'
            }
        }

        stage("Build Artifact"){
            steps{
                sh 'mvn clean install'
            }
        }
        stage("Static Code analysis"){
            steps{ 
                script{
                    withSonarQubeEnv(credentialsId: 'sonar-api-key') {
                    sh "mvn clean package sonar:sonar"
                    }
                }
            }
        }
        stage("Quality Gate Status"){
            steps{
                script{
                    waitForQualityGate abortPipeline: false, credentialsId: 'sonar-api-key'
                }
            }
        }
        stage ("Upload file to nexus"){
            steps{
                script{
                    def readPomVersion = readMavenPom file:'pom.xml'

                    def nexusRepo = readPomVersion.version.endsWith("SNAPSHOT") ? "maven-repo-snapshot": "maven-repo-release"
                    
                    nexusArtifactUploader artifacts: [
                        [
                            artifactId: "${readPomVersion.artifactId}", 
                        classifier: '', 
                        file: 'target/Toluxbuild.war', 
                        type: 'war']
                        ], 
                        credentialsId: 'Nexus2-login', 
                        groupId: "${readPomVersion.groupId}", 
                        nexusUrl: '172.21.0.2:8081/nexus', 
                        nexusVersion: 'nexus2', 
                        protocol: 'http', 
                        repository: nexusRepo, 
                        version: "${readPomVersion.version}"
                }
            }
        }
    }
}
