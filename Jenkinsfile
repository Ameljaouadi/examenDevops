pipeline {
    agent any
    tools {
        jdk 'jdk'
        nodejs 'Nodejs'
    }
    environment {
        SCANNER_HOME = tool 'sonar-server'
        dockerImageName = 'amel91/expensse-tracker2'
        dockerImage = ""
    }
    stages {
        stage('Workspace Cleaning') {
            steps {
                cleanWs()
            }
        }
        stage('Checkout from Git') {
            steps {
                git branch: 'main', url: 'https://github.com/Ameljaouadi/expensse-tracker2.git'
            }
        }
         stage("Sonarqube Analysis"){
         steps{
        withSonarQubeEnv('sonar-server') {
            sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=expensse-tracker2 \
            -Dsonar.projectKey=expensse-tracker2 \
            '''
        }
         }
          }
	
        stage('Install Dependencies') {
            steps {
                sh "npm install"
            }
        }
        stage('Build image') {
            steps {
                script {
                    dockerImage = docker.build dockerImageName
                }
            }
        }
        stage('Pushing Image') {
            environment {
                registryCredential = credentials('dockerhub')
            }
            steps {
                script {
                    docker.withRegistry('', registryCredential) {
                        dockerImage.push('latest')
                    }
                }
            }
        }
        
        stage('Deploy to Kubernetes'){
            steps{
                script{
                    dir('Kubernetes') {
                        withKubeConfig(caCertificate: '', clusterName: '', contextName: '', credentialsId: 'k8s', namespace: '', restrictKubeConfigAccess: false, serverUrl: '') {
                                sh 'kubectl apply -f deployment.yml'
                                sh 'kubectl apply -f service.yml'
                                sh 'kubectl get svc'
                                sh 'kubectl get all'
                        }   
                    }
                }
            }
        }
    }
        
    post {
        always {
            // Clean up Minikube after the pipeline finishes
            sh 'minikube stop'
        }
    }
}
