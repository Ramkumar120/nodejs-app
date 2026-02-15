pipeline {
    agent any
    
    environment {
        SCANNER_HOME = tool 'sonar-scanner'
        DOCKER_IMAGE = "ramkumar1999/my-nodejs:${env.BUILD_NUMBER}"
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
                    sh ''' 
                        ${SCANNER_HOME}/bin/sonar-scanner \
                        -Dsonar.projectName=my-nodejs \
                        -Dsonar.projectKey=my-nodejs
                    '''
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
                            echo "$password" | docker login -u ${username} --password-stdin
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
        stage('Update manifest repo') {
            steps {
                 withCredentials([usernamePassword(credentialsId: 'git-cred', passwordVariable: 'gitpassword', usernameVariable: 'gitusername')]) {
                  sh "git clone https://github.com/Ramkumar120/argocd.git"
                    sh '''
                    cd argocd/my-app
                    git config user.email 'jenkins@example.com'
                    git config user.name 'jenkins'
                    cat deployment.yaml
                    sed -i "s|image: ramkumar1999/my-nodejs:.*|image: ramkumar1999/my-nodejs:${BUILD_NUMBER}|" deployment.yaml
                    git add deployment.yaml
                    git commit -m "update image tag to $BUILD_NUMBER"
                    git push https://${gitusername}:${gitpassword}@github.com/Ramkumar120/argocd.git HEAD:main
                    '''
                 }
            }
        }
    }
}
