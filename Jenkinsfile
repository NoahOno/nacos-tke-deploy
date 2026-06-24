pipeline {
  agent any

  parameters {
    string(
      name: 'BRANCH',
      defaultValue: 'main',
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
          sh '''
            set -e

            NAMESPACE="${NACOS_NAMESPACE:-nacos}"

            if [ -z "${NACOS_AUTH_TOKEN}" ] || [ -z "${NACOS_AUTH_IDENTITY_KEY}" ] || [ -z "${NACOS_AUTH_IDENTITY_VALUE}" ]; then
              echo "ERROR: 请在 CODING 项目变量中配置 NACOS_AUTH_TOKEN / NACOS_AUTH_IDENTITY_KEY / NACOS_AUTH_IDENTITY_VALUE"
              exit 1
            fi

            kubectl apply -f k8s/tke/namespace.yaml
            sleep 3

            kubectl create secret generic nacos-auth -n "${NAMESPACE}" \
              --from-literal=auth-token="${NACOS_AUTH_TOKEN}" \
              --from-literal=identity-key="${NACOS_AUTH_IDENTITY_KEY}" \
              --from-literal=identity-value="${NACOS_AUTH_IDENTITY_VALUE}" \
              --dry-run=client -o yaml | kubectl apply -f -

            kubectl apply -f k8s/tke/configmap.yaml
            kubectl apply -f k8s/tke/service.yaml
            kubectl apply -f k8s/tke/deployment.yaml

            kubectl rollout status deployment/nacos -n "${NAMESPACE}" --timeout=300s
            kubectl get pods,svc -n "${NAMESPACE}" -l app=nacos
          '''
        }
      }
    }
  }
}
