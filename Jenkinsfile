  pipeline {
    agent any
  
    parameters {
      string(
        name: 'BRANCH',
        defaultValue: 'master',
        description: '要部署的分支'
      )
    }
  
    stages {
      stage('检出') {
        steps {
          checkout([$class: 'GitSCM',
            branches: [[name: params.BRANCH]],
            userRemoteConfigs: [[
              url: env.GIT_REPO_URL,
              credentialsId: env.CREDENTIALS_ID
            ]]
          ])
        }
      }
  
      stage('部署 TKE') {
        steps {
          withKubeConfig([credentialsId: env.TKE_CLUSTER_CREDENTIAL_ID]) {
            sh "kubectl apply -f k8s/tke/namespace.yaml && sleep 3"
            sh "kubectl apply -f k8s/tke/"
          }
        }
      }
    }
  }
