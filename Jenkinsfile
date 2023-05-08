pipeline {
  agent {
    kubernetes {
      yaml """
kind: Pod
metadata:
  name: kaniko
spec:
  containers:
  - name: kubectl
    image: bearengineer/awscli-kubectl
    command:
    - /bin/cat
    tty: true    
  - name: kaniko
    image: gcr.io/kaniko-project/executor:debug
    command:
    - /busybox/cat
    tty: true
    volumeMounts:
      - name: kaniko-secret
        mountPath: /kaniko/.docker
  volumes:
    - name: kaniko-secret
      secret:
        secretName: docker-credentials
        items:
          - key: .dockerconfigjson
            path: config.json
        """
    }
  }
  stages {
    stage('Checkout Source') {
        steps {
            git branch: 'main',
            credentialsId: 'prashantdh28',
            url: 'https://github.com/prashantdh28/demo.git'
        }
    }
    stage('Kaniko Build & Push Image') {
        steps {
            container('kaniko') {
                script {
                    sh '''
                    /kaniko/executor --dockerfile `pwd`/Dockerfile \
                                     --context `pwd` \
                                     --destination=prashant28/image-1:${BUILD_NUMBER}
                    '''
          }
        }
      }
    }
    stage('Deploy App to Kubernetes') {     
      steps {
        container('kubectl') {
          withCredentials([string(credentialsId: 'awscred', variable: 'AWS_ACCESS_KEY_ID'), string(credentialsId: 'awscred', variable: 'AWS_SECRET_ACCESS_KEY')]){
            sh 'sh 'aws eks update-kubeconfig --name EKS-CLUSTER --region us-east-2'
          withCredentials([file(credentialsId: 'kubeconfig', variable: 'KUBECONFIG')]) {
            sh 'export KUBECONFIG=config'
            sh 'kubectl version'
            sh 'cat deployment.yaml'
            sh 'kubectl apply -f deployment.yaml'
          }
          }
        }
      }
    }
  }
}
