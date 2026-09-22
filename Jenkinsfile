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
                    sh 'docker build -t registry.cn-guangzhou.aliyuncs.com/etlbat/demo:latest .'
                }
            }
        }

        stage('Docker Push') {
            steps {
                echo '🚀 推送镜像到仓库...'
                sh 'docker push registry.cn-guangzhou.aliyuncs.com/etlbat/demo:latest'
            }
        }

        stage('Deploy') {
            steps {
                echo '📦 部署应用...'
                sh '''
                    docker stop demo-app 2>/dev/null || true
                    docker rm demo-app 2>/dev/null || true
                    docker run -d --name demo-app -p 8082:8080 registry.cn-guangzhou.aliyuncs.com/etlbat/demo:latest
                '''
            }
        }

        stage('Commit Build Info') {
            steps {
                echo '📝 提交构建信息...'
                dir('demo') {
                    sh '''
                        git config user.name "jenkins"
                        git config user.email "jenkins@ci.local"
                        echo "Last build: ${BUILD_NUMBER} - $(date '+%Y-%m-%d %H:%M:%S')" > build-info.txt
                        git add build-info.txt
                        git diff --cached --quiet || git commit -m "ci: update build info #${BUILD_NUMBER}"
                        git push origin HEAD:main
                    '''
                }
            }
        }
    }

    post {
        success { echo '✅ 构建成功！' }
        failure { echo '❌ 构建失败！' }
        always  { cleanWs() }
    }
}
