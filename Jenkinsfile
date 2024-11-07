pipeline {
    agent any

	environment{
		DOCKER_IMAGE_OWNER = 'nanakia1031'
		DOCKER_IMAGE_TAG = "v${BUILD_NUMBER}" // 매 빌드마다 고유 태그 생성
		DOCKER_TOKEN = credentials('dockerhub')
	}

    stages {
        stage('clone from SCM') {
            steps {
                sh '''
                rm -rf project-jenkins
				git clone https://github.com/cks1031/project-jenkins.git
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
        stage('Trigger ArgoCD') {
            steps {
                script {
                    sh '''
                    curl -k -X POST https://43.202.101.98:30765/api/webhook \
                        -H "Content-Type: application/json" \
                        -d '{
                            "ref": "refs/heads/main",
                            "before": "abcdef1234567890abcdef1234567890abcdef1234",  # 실제 이전 커밋 ID
                            "after": "123456abcdef1234567890abcdef12345678901234",  # 현재 커밋 ID
                            "repository": {
                                "name": "project-argocd",
                                "url": "https://github.com/cks1031/project-argocd.git"
                            },
                            "pusher": {
                                "name": "jenkins",
                                "email": "jenkins@jenkins.com"
                            }
                        }'
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