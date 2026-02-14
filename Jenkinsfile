pipeline {
    agent any
    
    environment {
        SCANNER_HOME = tool 'sonar-scanner'
        DOCKER_IMAGE = 'ramkumar1999/my-nodejs:latest'
        CLUSTER = 'my-cluster'
        REGION = 'us-east-1'
        
    }
    
    tools {
        nodejs 'nodejs'
    }

    stages {
        stage('clean workspace') {
            steps {
                cleanWs()
            }
        }
        stage('Git checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/Ramkumar120/nodejs-app.git'
            }
        }
        stage('Install dependencies') {
            steps {
                sh "npm install"
            }
        }
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonar-qube') {
                    sh """ 
                    ${env.SCANNER_HOME}/bin/sonar-scanner \
                        -Dsonar.projectName=my-nodejs \
                        -Dsonar.projectKey=my-nodejs
                    """
                }
            }
        }
        stage('Quality Gate') {
            steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'sonar-token'
                }
            }
        }
        stage('Build Docker Image') {
            steps {
                script{
                   withCredentials([usernamePassword(credentialsId: 'docker-token', passwordVariable: 'password', usernameVariable: 'username')])  {
                        sh '''
                            echo "${password}" | docker login -u ${username} --password-stdin
                            docker build -t ${DOCKER_IMAGE} .
                            docker push ${DOCKER_IMAGE}
                        '''
                    }
                }
            }
        }
         stage('Trivy scan') {
            steps {
                sh "trivy image ${DOCKER_IMAGE}"
            }
        }
        stage('Deploying to eks cluster') {
            steps {
                sh """
                    aws eks update-kubeconfig --name ${env.CLUSTER} --region ${env.REGION}
                    kubectl get nodes
                    kubectl apply -f deployment.yaml
                    kubectl apply -f service.yaml
                """
            }
        }
    }
}
