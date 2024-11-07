pipeline {
    agent any

	environment{
		DOCKER_IMAGE_OWNER = 'nanakia1031'
		DOCKER_IMAGE_TAG = "v${BUILD_NUMBER}" // 매 빌드마다 고유 태그 생성
		DOCKER_TOKEN = credentials('dockerhub')
		ARGOCD_WEBHOOK_URL = 'https://43.202.101.98:30765/api/webhook'  // ArgoCD Webhook URL
	}

    stages {
        stage('clone from SCM') {
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
		stage('Docker Image pushing') {
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
                sed -i 's|image: {{.Values.image.admin.repository}}:{{.Values.image.admin.tag}}|image: ${DOCKER_IMAGE_OWNER}/prj-admin:${DOCKER_IMAGE_TAG}|g' project-argocd/deploy-argocd/templates/deployment.yaml
                sed -i 's|image: {{.Values.image.visitor.repository}}:{{.Values.image.visitor.tag}}|image: ${DOCKER_IMAGE_OWNER}/prj-visitor:${DOCKER_IMAGE_TAG}|g' project-argocd/deploy-argocd/templates/deployment.yaml
                sed -i 's|image: {{.Values.image.frontend.repository}}:{{.Values.image.frontend.tag}}|image: ${DOCKER_IMAGE_OWNER}/prj-frontend:${DOCKER_IMAGE_TAG}|g' project-argocd/deploy-argocd/templates/deployment.yaml
                '''
            }
        }
        stage('Commit and Push ArgoCD Deployment YAML Changes') {
            steps {
                dir('project-argocd') {
                    sh '''
                    git config user.name "cks1031"
                    git config user.email "cks1031@naver.com"
                    git add deploy-argocd/templates/deployment.yaml
                    git commit -m "Update image tags to ${DOCKER_IMAGE_TAG}"
                    git push origin main
                    '''
                }
            }
        }

        stage('Trigger ArgoCD') {
            steps {
                sh '''
                curl -k -X POST ${ARGOCD_WEBHOOK_URL}
                '''
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