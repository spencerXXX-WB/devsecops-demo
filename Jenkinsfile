pipeline {
    agent any
    environment {
      GIT_CREDENTIALS_ID = 'Git-credentials'
      GIT_USER_EMAIL = 'office@spencer.com'
      GIT_USER_NAME = 'JohnSpencer'
     }
    parameters {
        string defaultValue: 'latest', description: 'Docker tag', name: 'DOCKER_TAG', trim: true
    }
    stages {
        stage('Install') {
            steps {
                 nodejs('node20') {
                    sh 'npm install'
                 }
            }
        }
        stage('build') {
            steps {
                 nodejs('node20') {
                    sh 'npm run build'
                 }
            }
        }
        stage('image-build') {
            steps {
                 script{
                    withDockerRegistry(credentialsId: 'docker-credentials') {
                       sh " docker build -t spencerxxxwb/tictac:${params.DOCKER_TAG} ." 
                    }
                 }
            }
        }
        stage('image-scan') {
            steps {
                sh "trivy image --format table -o output.html spencerxxxwb/tictac:${params.DOCKER_TAG} "
            }
        }
        stage('image-push') {
            steps {
                 script{
                    withDockerRegistry(credentialsId: 'docker-credentials') {
                       sh " docker push spencerxxxwb/tictac:${params.DOCKER_TAG} "
                    }
                 }
            }
        }
        stage('update-manifest') {
            steps {
                 dir('kubernetes') {
                    sh '''
                        sed -i 's|image: spencerxxxwb/tictac[^ ]*|image: spencerxxxwb/tictac:'"${DOCKER_TAG}"'|' deployment.yaml
                        
                    '''
                }
            }
        }
        stage('Push Updated Deployment Files') {
            steps {
                withCredentials([gitUsernamePassword(credentialsId: GIT_CREDENTIALS_ID, gitToolName: 'Default')]) {
                    dir('kubernetes') {
                        sh '''
                            git config user.email "${GIT_USER_EMAIL}"
                            git config user.name "${GIT_USER_NAME}"
                            git add deployment.yaml
                            git status
                            git commit -m "Update image tag to ${DOCKER_TAG}"
                            git push origin main
                        '''
                    }
                }
            }
        }  
    }
}
