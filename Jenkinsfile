pipeline{
    agent any
    tools {
        jdk 'jdk17'
        maven 'maven3'
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
        stage ('Maven Compile') {
            steps {
                sh 'mvn clean verify'
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
   }
}