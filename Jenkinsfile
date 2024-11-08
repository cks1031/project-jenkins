pipeline {
    agent any

    environment {
        DOCKER_IMAGE_OWNER = 'nanakia1031'
        DOCKER_BUILD_TAG = "v${env.BUILD_NUMBER}"
        DOCKER_TOKEN = credentials('dockerhub')
        GIT_CREDENTIALS = credentials('github_token')
        REPO_URL = 'cks1031/project-jenkins.git'
        ARGOCD_REPO_URL = 'cks1031/project-argocd.git'
        COMMIT_MESSAGE = 'Update README.md via Jenkins Pipeline'
    }

    stages {
            stage('Clone from SCM') {
                steps {
                    dir('project-jenkins') {
                        git branch: 'master', url: "https://${GIT_CREDENTIALS_USR}:${GIT_CREDENTIALS_PSW}@github.com/${REPO_URL}"
                    }
                    dir('project-argocd') {
                        git branch: 'master', url: "https://${GIT_CREDENTIALS_USR}:${GIT_CREDENTIALS_PSW}@github.com/${ARGOCD_REPO_URL}"
                    }
            }
        }
    stage('Docker Image Building') {
          steps {
                sh '''
                cd project-jenkins
                docker build -t ${DOCKER_IMAGE_OWNER}/prj-frontend:${DOCKER_BUILD_TAG} ./frontend
                docker build -t ${DOCKER_IMAGE_OWNER}/prj-admin:${DOCKER_BUILD_TAG} ./admin-service
                docker build -t ${DOCKER_IMAGE_OWNER}/prj-visitor:${DOCKER_BUILD_TAG} ./visitor-service
                '''
            }
        }

        stage('Docker Login') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub', usernameVariable: 'DOCKER_USR', passwordVariable: 'DOCKER_PWD')]) {
                    sh "echo $DOCKER_PWD | docker login -u $DOCKER_USR --password-stdin"
                }
            }
        }

        stage('Docker Image Pushing') {
            steps {
                sh '''
                docker push ${DOCKER_IMAGE_OWNER}/prj-frontend:${DOCKER_BUILD_TAG}
                docker push ${DOCKER_IMAGE_OWNER}/prj-admin:${DOCKER_BUILD_TAG}
                docker push ${DOCKER_IMAGE_OWNER}/prj-visitor:${DOCKER_BUILD_TAG}
                '''
            }
        }

        stage('Update ArgoCD Deployment YAML with Image Tags') {
            steps {
                sh '''
                sed -i "s|image: {{.Values.image.admin.repository}}|image: ${DOCKER_IMAGE_OWNER}/prj-admin:${DOCKER_BUILD_TAG}|g" deploy-argocd/templates/deployment.yaml
                sed -i "s|image: {{.Values.image.frontend.repository}}|image: ${DOCKER_IMAGE_OWNER}/prj-frontend:${DOCKER_BUILD_TAG}|g" deploy-argocd/templates/deployment.yaml
                sed -i "s|image: {{.Values.image.visitor.repository}}|image: ${DOCKER_IMAGE_OWNER}/prj-visitor:${DOCKER_BUILD_TAG}|g" deploy-argocd/templates/deployment.yaml
                '''
            }
        }

        stage('Commit Changes') {
            steps {
                dir('project-argocd') {
                    sh '''
                    git config user.name "cks1031"
                    git config user.email "cks1031@jenkins.com"
                    git add deploy-argocd/templates/deployment.yaml
                    git commit -m "Update image tags to ${DOCKER_BUILD_TAG}"
                    git push https://${GIT_CREDENTIALS_USR}:${GIT_CREDENTIALS_PSW}@github.com/${ARGOCD_REPO_URL} main
                    '''
                }
            }
        }

        stage('Docker Logout') {
            steps {
                sh '''
                docker logout
                '''
            }
        }
    }
}
