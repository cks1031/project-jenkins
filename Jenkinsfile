pipeline {
    agent any

    environment {
        DOCKER_IMAGE_OWNER = 'nanakia1031'
        DOCKER_IMAGE_TAG = "v${BUILD_NUMBER}" // unique tag for each build
        DOCKER_TOKEN = credentials('dockerhub')
        GIT_CREDENTIALS = credentials('github_token')
        REPO_URL = 'cks1031/project-argocd.git'
    }

    stages {
        stage('Clone from SCM') {
            steps {
                sh '''
                rm -rf project-jenkins project-argocd
                git clone https://github.com/cks1031/project-jenkins.git
                git clone https://github.com/cks1031/project-argocd.git
                '''
            }
        }

        stage('Docker Image Building') {
            steps {
                sh '''
                cd project-jenkins
                docker build -t ${DOCKER_IMAGE_OWNER}/prj-frontend:${DOCKER_IMAGE_TAG} ./frontend
                docker build -t ${DOCKER_IMAGE_OWNER}/prj-admin:${DOCKER_IMAGE_TAG} ./admin-service
                docker build -t ${DOCKER_IMAGE_OWNER}/prj-visitor:${DOCKER_IMAGE_TAG} ./visitor-service
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
                docker push ${DOCKER_IMAGE_OWNER}/prj-frontend:${DOCKER_IMAGE_TAG}
                docker push ${DOCKER_IMAGE_OWNER}/prj-admin:${DOCKER_IMAGE_TAG}
                docker push ${DOCKER_IMAGE_OWNER}/prj-visitor:${DOCKER_IMAGE_TAG}
                '''
            }
        }

        stage('Update ArgoCD Deployment YAML with Image Tags') {
            steps {
                sh '''
                sed -i 's|image: {{ .Values.image.admin.repository }}|image: nanakia1031/prj-admin:{{.Values.image.admin.tag}}|g' project-argocd/deploy-argocd/templates/deployment.yaml
                sed -i 's|image: {{ .Values.image.frontend.repository }}|image: nanakia1031/prj-frontend:{{.Values.image.visitor.tag}}|g' project-argocd/deploy-argocd/templates/deployment.yaml
                sed -i 's|image: {{ .Values.image.visitor.repository }}|image: nanakia1031/prj-visitor:{{.Values.image.frontend.tag}}|g' project-argocd/deploy-argocd/templates/deployment.yaml
                '''
            }
        }

        stage('Commit and Push ArgoCD Deployment YAML Changes') {
            steps {
                dir('project-argocd') {
                    sh '''
                    git config user.name "cks1031"
                    git config user.email "cks1031@jenkins.com"
                    git add deploy-argocd/templates/deployment.yaml
                    git commit -m "Update image tags to ${DOCKER_IMAGE_TAG}"
                    git push https://${GIT_CREDENTIALS_USR}:${GIT_CREDENTIALS_PSW}@github.com/${REPO_URL} master
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
