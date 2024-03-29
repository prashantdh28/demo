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
                                     --destination=prashant28/image-1
                    '''
          }
        }
      }
    }
    stage('Deploy App to Kubernetes') {     
      steps {
        container('kubectl') {
            withCredentials([file(credentialsId: 'kubeconfig', variable: 'KUBECONFIG')]) {
            sh 'aws configure set aws_access_key_id "AKIA4BSLOBW5YZGXOXHG" && aws configure set aws_secret_access_key "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"  && aws configure set region "us-east-2" && aws configure set output "json"'
            sh 'aws eks update-kubeconfig --name EKS-CLUSTER --region us-east-2'
            sh 'kubectl version'
            sh 'kubectl apply -f deployment.yaml'
          }
        }
      }
    }
  }
}
