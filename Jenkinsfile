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
    image: gcr.m.daocloud.io/kaniko-project/executor:debug
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
                      --destination=192.168.49.2:30500/hcd/wordpress:${BUILD_NUMBER} \
                      --insecure-registry=192.168.49.2:30500 \
                      --cache=true
                    '''
                }
            }
        }
        stage('部署到 K8s') {
            steps {
                container('kubectl') {
                    sh '''
                    export KUBECONFIG=/tmp/kubeconfig
                    kubectl config set-cluster in-cluster --server=https://kubernetes.default.svc --insecure-skip-tls-verify=true
                    kubectl config set-credentials jenkins-sa --token=$(cat /var/run/secrets/kubernetes.io/serviceaccount/token)
                    kubectl config set-context jenkins-ctx --cluster=in-cluster --user=jenkins-sa
                    kubectl config use-context jenkins-ctx
                    kubectl set image deployment/wordpress wordpress=192.168.49.2:30500/hcd/wordpress:${BUILD_NUMBER} -n devops
                    kubectl rollout status deployment/wordpress -n devops
                    '''
                }
            }
        }
    }
}
