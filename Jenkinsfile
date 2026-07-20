pipeline {
    agent any
    environment {
        REGISTRY = 'registry.registry.svc.cluster.local:5000'
        BACKEND_IMAGE = "${REGISTRY}/java-app-backend"
        FRONTEND_IMAGE = "${REGISTRY}/java-app-frontend"
        TAG = "${BUILD_NUMBER}"
        CHART_PATH = './charts'
        NAMESPACE = 'app'
        RELEASE_NAME = 'java-app'
    }

    stages {
        stage('Checkout') {
            agent any
            steps {
                checkout scm
                stash name: 'source', includes: '**'
            }
        }

        stage('Build Backend') {
            agent {
                kubernetes {
                    label 'kaniko-backend'
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
"""
                }
            }
            steps {
                unstash 'source'
                container('kaniko') {
                    sh '''
                        /kaniko/executor \
                            --context=./Java-app/back-end \
                            --dockerfile=Dockerfile \
                            --destination=${BACKEND_IMAGE}:${TAG} \
                            --cache=true \
                            --cache-repo=${REGISTRY}/java-app-backend-cache \
                            --use-new-run \
                            --cleanup
                    '''
                }
            }
        }

        stage('Build Frontend') {
            agent {
                kubernetes {
                    label 'kaniko-frontend'
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
"""
                }
            }
            steps {
                unstash 'source'
                container('kaniko') {
                    sh '''
                        /kaniko/executor \
                            --context=./Java-app/front-end \
                            --dockerfile=Dockerfile \
                            --destination=${FRONTEND_IMAGE}:${TAG} \
                            --cache=true \
                            --cache-repo=${REGISTRY}/java-app-frontend-cache \
                            --use-new-run \
                            --cleanup
                    '''
                }
            }
        }

        stage('Deploy with Helm') {
    agent {
        kubernetes {
            label 'helm-deploy'
            yaml """
spec:
  serviceAccountName: jenkins-deploy
  containers:
    - name: jnlp
      image: jenkins/inbound-agent:latest
    - name: helm
      image: alpine/helm:3.14.0
      command:
        - /bin/sleep
        - infinity
"""
        }
    }
    steps {
        unstash 'source'
        container('helm') {
            script {
                sh """
                    helm upgrade --install ${RELEASE_NAME} ${CHART_PATH} \
                        --namespace ${NAMESPACE} \
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