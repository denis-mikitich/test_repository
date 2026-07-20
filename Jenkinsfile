pipeline {
    agent {
        kubernetes {
            label 'kaniko-pod'
            podRetention 'always'   
            yaml """
spec:
  containers:
    - name: jnlp
      image: jenkins/inbound-agent:latest
    - name: kaniko
      image: gcr.io/kaniko-project/executor:debug
      command:
        - /busybox/sleep
        - infinity
      env:
        - name: PATH
          value: /busybox:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
    - name: kubectl
      image: bitnami/kubectl:latest
      command: ['/bin/sleep', 'infinity']
"""
        }
    }

    environment {
        REGISTRY = 'registry.registry.svc.cluster.local:5000'   
        BACKEND_IMAGE = "${REGISTRY}/java-app-backend"
        FRONTEND_IMAGE = "${REGISTRY}/java-app-frontend"
        TAG = "${BUILD_NUMBER}"                  // уникальный тег для каждого билда (номер сборки)
        CHART_PATH = './charts'               // путь к папке с Helm-чартом
        NAMESPACE = 'app'                        // namespace для деплоя
        RELEASE_NAME = 'java-app'                // имя релиза Helm
    }
    stages {
        stage('Checkout') {
            steps {
                checkout scm   
            }
        }

        stage('Build Backend') {
            steps {
                container('kaniko') {   
                    sh '''
                        /kaniko/executor \
                            --context=./Java-app/back-end \
                            --dockerfile=Dockerfile \
                            --destination=${BACKEND_IMAGE}:${TAG} \
                            --cache=true \
                            --cache-repo=${REGISTRY}/java-app-backend-cache \
                            --use-new-run
                    '''
                }
            }
        }

        stage('Build Frontend') {
            steps {
                container('kaniko') {
                    sh '''
                        /kaniko/executor \
                            --context=./Java-app/front-end \
                            --dockerfile=Dockerfile \
                            --destination=${FRONTEND_IMAGE}:${TAG} \
                            --cache=true \
                            --cache-repo=${REGISTRY}/java-app-frontend-cache \
                            --use-new-run
                    '''
                }
            }
        }

        stage('Deploy with Helm') {
            steps {
                container('kubectl') {
                    script {
                        sh """
                            helm upgrade --install ${RELEASE_NAME} ${CHART_PATH} \
                                --namespace ${NAMESPACE} --create-namespace \
                                --set backend.image.tag=${TAG} \
                                --set frontend.image.tag=${TAG} \
                                --wait
                        """
                    }
                }
            }
        }
    }

    post {
        success {
            echo 'Деплой успешно выполнен! Приложение доступно по адресу http://app.local'
        }
        failure {
            echo 'Сборка или деплой завершились с ошибкой. Проверьте логи выше.'
        }
    }
}
