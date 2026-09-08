pipeline {
    agent {
        kubernetes {
            yaml """
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: jnlp
    image: docker.m.daocloud.io/jenkins/inbound-agent:latest
    imagePullPolicy: IfNotPresent
  - name: kaniko
    image: docker.m.daocloud.io/gcr.io/kaniko-project/executor:debug
    imagePullPolicy: IfNotPresent
    command: ["sleep"]
    args: ["infinity"]
    volumeMounts:
    - name: docker-config
      mountPath: /kaniko/.docker/config.json
      subPath: config.json
  - name: kubectl
    image: docker.m.daocloud.io/bitnami/kubectl:latest
    imagePullPolicy: IfNotPresent
    command: ["sleep"]
    args: ["infinity"]
  volumes:
  - name: docker-config
    secret:
      secretName: acr-credentials
      items:
      - key: .dockerconfigjson
        path: config.json
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
