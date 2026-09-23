pipeline{
    agent any
    tools {
        jdk 'jdk17'
        maven 'maven3'
    }
    environment {
        DOCKER_IMAGE: "cyber0ps/petstore"
    }
    stages{
        stage ('Clean Workspace'){
            steps{
                cleanWs()
            }
        }
        stage ('Checkout SCM') {
            steps {
                git branch: 'master', url: 'https://github.com/cyberops-homelab/jpetstore.git'
            }
        }
        stage ('Maven Build and Test') {
            steps {
                sh 'mvn clean verify DskipTests=true'
            }
        }
        stage("Sonarqube Analysis "){
            steps{
                withSonarQubeEnv('sonar-server') {
                    sh ''' mvn org.sonarsource.scanner.maven:sonar-maven-plugin:sonar -Dsonar.projectName=PetStore \
                    -Dsonar.projectKey=PetStore '''
                }
            }
        }
        stage("Quality Gate"){
            steps {
                script {
                  waitForQualityGate abortPipeline: true, credentialsId: 'sonar-token'
                }
           }
        }
        stage('Docker Build Image'){
            steps{
                sh '''
                    docker build -t ${DOCKER_IMAGE}:${BUILD_NUMBER} .
                '''
            }
        }
        stage("TRIVY Scan Image"){
            steps{
                sh "trivy image ${DOCKER_IMAGE}:${BUILD_NUMBER} > trivy.txt"
            }
        }
        stage("Docker Push Image"){
            steps{
                withDockerRegistry(credentialsId: 'docker-hub'){
                    sh "docker push ${DOCKER_IMAGE}:${BUILD_NUMBER}"
                }
            }
        }
        
   }
}