pipeline {
    agent any
    tools{
        jdk 'jdk17'
        maven 'maven3'
    }
    environment {
        SCANNER_HOME = tool 'sonar-scanner'
    }

    stages {
        stage('Git checkout') {
            steps {
                git branch: 'main', url:'https://github.com/mohitkumar1313/Boardgame.git'
            }
        }
        stage('Complie') {
            steps {
                sh "mvn compile"
            }
        }
        stage('Test') {
            steps {
                sh "mvn test"
            }
        }
        stage('Sonar Analysis'){
            steps{
                withSonarQubeEnv('sonar') {
                    sh '''$SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=BoardGame -Dsonar.projectKey=BoardGame \
                            -Dsonar.java.binaries=. '''
                }
            }
        }
        stage('Build') {
            steps {
                sh "mvn clean package"
            }
        
        }
        stage('Publish to Nexus'){
            steps{
                withMaven(globalMavenSettingsConfig: 'global-settings', jdk: 'jdk17', maven: 'maven3', mavenSettingsConfig: '', traceability: true) {
                    sh "mvn deploy"
                }
            }
        }
        stage('Build the Image') {
            steps{
                withDockerRegistry( url: 'https://index.docker.io/v1/', credentialsId: 'docker-cred') {
                    sh '''
                        docker build -t mohitsalgotra/boardgame:latest . 
                        docker push mohitsalgotra/boardgame:latest
                    '''
                } 
            }    
            
        }
        stage('Authenticate with GKE') {
            steps {
                script {
                    withCredentials([file(credentialsId: 'k8-cred', variable: 'GOOGLE_APPLICATION_CREDENTIALS')]) {
                        sh """
                            gcloud auth activate-service-account --key-file=$GOOGLE_APPLICATION_CREDENTIALS
                            gcloud config set project majestic-hybrid-435221-k7
                            gcloud container clusters get-credentials cluster-1 --zone us-central1-c
                        """
                    }
                }
            }
        }
        stage ('Deploy to cluster'){
            steps{
                script{
                    sh '''
                        kubectl apply -f deployment-service.yaml
                    '''
                }
            }
        }
        stage ('Getting pods') {
            steps{
                script{
                    sh ''' 
                        kubectl get pods
                        kubectl get svc
                    '''
                }
            }
        }
        
    }
}
