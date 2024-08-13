pipeline {
    agent any

    // 환경 변수 설정
    environment {
        DOCKER_REGISTRY = "neocloudacr001.azurecr.io"
        IMAGE_NAME = "demo-app"
        AZURE_CREDENTIALS_ID = "acr-credentials"
        BUILD_NUMBER = "${env.BUILD_NUMBER}"
        AZURE_TENANT_ID = "97f42f55-f1db-4804-b1eb-08db083efd4f"
    }

    stages {
        // Maven을 사용한 애플리케이션 빌드 단계
        stage('Build') {
            steps {
                sh 'mvn clean package'
            }
        }

        // Docker 이미지 빌드 단계
        stage('Build Docker Image') {
            steps {
                script {
                    sh "docker build -t ${DOCKER_REGISTRY}/${IMAGE_NAME}:${BUILD_NUMBER} ."
                }
            }
        }

        // Azure Container Registry(ACR)에 Docker 이미지 푸시 단계
        stage('Push Docker Image') {
            steps {
                script {
                    // Azure 자격 증명을 사용하여 ACR에 로그인 및 이미지 푸시
                    withCredentials([usernamePassword(credentialsId: AZURE_CREDENTIALS_ID, passwordVariable: 'AZURE_PASSWORD', usernameVariable: 'AZURE_USER')]) {
                        // Azure CLI를 사용하여 서비스 주체로 로그인
                        sh 'az login --service-principal -u ${AZURE_USER} -p ${AZURE_PASSWORD} --tenant ${AZURE_TENANT_ID}'
                        // ACR에 로그인
                        sh "az acr login --name ${DOCKER_REGISTRY}"
                        // Docker 이미지를 ACR에 푸시
                        sh "docker push ${DOCKER_REGISTRY}/${IMAGE_NAME}:${BUILD_NUMBER}"
                    }
                }
            }
        }

        // Azure Kubernetes Service(AKS)에 애플리케이션 배포 단계
        stage('Deploy to AKS') {
            steps {
                script {
                    // deployment.yaml 파일의 이미지 태그를 현재 빌드 번호로 업데이트
                    sh "sed -i 's|image: .*|image: ${DOCKER_REGISTRY}/${IMAGE_NAME}:${BUILD_NUMBER}|' deployment.yaml"
                    // Kubernetes 배포 적용
                    sh "kubectl apply -f deployment.yaml"
                    // Kubernetes 서비스 적용
                    sh "kubectl apply -f service.yaml"
                    // 배포 상태 확인
                    sh "kubectl rollout status deployment/${IMAGE_NAME}"
                }
            }
        }
    }
}
