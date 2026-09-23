pipeline {
    agent any

    environment {
        REPO_URL = 'git@github.com:BluesABC/demo.git'
        BRANCH = 'main'
    }

    stages {
        stage('Checkout') {
            steps {
                echo '📥 拉取代码...'
                checkout([
                    $class: 'GitSCM',
                    branches: [[name: "*/${BRANCH}"]],
                    userRemoteConfigs: [[url: REPO_URL]]
                ])
            }
        }

        stage('Build') {
            steps {
                echo '🔨 Maven 编译打包...'
                dir('demo') {
                    sh 'chmod +x mvnw && ./mvnw -B clean package -DskipTests'
                }
            }
        }

        stage('Test') {
            steps {
                echo '🧪 运行单元测试...'
                dir('demo') {
                    sh './mvnw -B test'
                }
            }
            post {
                always {
                    junit 'demo/target/surefire-reports/*.xml'
                }
            }
        }

        stage('Docker Build') {
            steps {
                echo '🐳 构建 Docker 镜像...'
                dir('demo') {
                    sh 'DOCKER_BUILDKIT=0 docker build -t crpi-n4a8umbyx0cmpkk6.cn-guangzhou.personal.cr.aliyuncs.com/etlbat/demo:latest .'
                }
            }
        }

        stage('Docker Login') {
            steps {
                echo '🔑 登录阿里云镜像仓库...'
                withCredentials([usernamePassword(credentialsId: 'aliyun-acr', usernameVariable: 'ACR_USER', passwordVariable: 'ACR_PASS')]) {
                    sh 'echo "USER: $ACR_USER"'
                    sh "echo \$ACR_PASS | docker login --username=\$ACR_USER --password-stdin crpi-n4a8umbyx0cmpkk6.cn-guangzhou.personal.cr.aliyuncs.com"
                }
            }
        }

        stage('Docker Push') {
            steps {
                echo '🚀 推送镜像到仓库...'
                sh 'docker push crpi-n4a8umbyx0cmpkk6.cn-guangzhou.personal.cr.aliyuncs.com/etlbat/demo:latest'
            }
        }

        stage('Deploy') {
            steps {
                echo '📦 部署应用...'
                sh '''
                    docker stop demo-app 2>/dev/null || true
                    docker rm demo-app 2>/dev/null || true
                    docker run -d --name demo-app -p 8082:8080 crpi-n4a8umbyx0cmpkk6.cn-guangzhou.personal.cr.aliyuncs.com/etlbat/demo:latest
                '''
            }
        }

    }

    post {
        success { echo '✅ 构建成功！' }
        failure { echo '❌ 构建失败！' }
        always  { cleanWs() }
    }
}
