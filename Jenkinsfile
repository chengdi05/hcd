pipeline {
    agent {
        kubernetes {
            yaml """
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: jnlp
    image: jenkins/inbound-agent:latest
  - name: kaniko
    image: gcr.io/kaniko-project/executor:debug
    command: ["sleep"]
    args: ["infinity"]
    volumeMounts:
    - name: docker-config
      mountPath: /kaniko/.docker/
  - name: kubectl
    image: bitnami/kubectl:latest
    command: ["sleep"]
    args: ["infinity"]
  volumes:
  - name: docker-config
    secret:
      secretName: acr-credentials
"""
        }
    }
    stages {
        stage('拉取代码') {
            steps { 
                checkout scm 
            }
        }
        stage('构建镜像并推送 ACR') {
            steps {
                container('kaniko') {
                    sh '''
                    /kaniko/executor \
                      --dockerfile=Dockerfile \
                      --context=dir://${WORKSPACE} \
                      --destination=registry.cn-hangzhou.aliyuncs.com/hcd05/wordpress:${BUILD_NUMBER} \
                      --cache=true
                    '''
                }
            }
        }
        stage('部署到 K8s') {
            steps {
                container('kubectl') {
                    sh '''
                    kubectl set image deployment/wordpress wordpress=registry.cn-hangzhou.aliyuncs.com/hcd05/wordpress:${BUILD_NUMBER} -n devops
                    kubectl rollout status deployment/wordpress -n devops
                    '''
                }
            }
        }
    }
}
