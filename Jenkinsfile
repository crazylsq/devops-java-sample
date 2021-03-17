pipeline {
  agent {
    kubernetes {
      cloud 'kubernetes'
      label 'maven-openjdk8'
    }
  }

  environment {
      DOCKER_CREDENTIAL_ID = 'TKE-TCR'
      REGISTRY = 'test-sino.tencentcloudcr.com'
      DOCKERHUB_NAMESPACE = 'sino'
      APP_NAME = 'devops-java-sample'
  }

  stages {
      stage ('clone k8s yaml') {
        steps {
          dir ('deploy') {
            git url: "https://github.com/crazylsq/yaml.git", branch:'master'
          }
        }
      }

      stage ('prepare') {
        steps {
          withCredentials([file(credentialsId: 'config', variable: 'KUBECONFIG')]) {
            sh "mkdir -p ${env.WORKSPACE}/.kube && cp ${KUBECONFIG} ${env.WORKSPACE}/.kube/config"
          }
        }
      }

      stage ('git commit hash') {
        steps {
          script {
              env.GIT_HASH = sh (script: 'git rev-parse --short HEAD', returnStdout: true).trim()
          }
        }
      }

      stage ('build & push') {
          steps {
              container ('maven-openjdk8') {
                  sh 'mvn -o -Dmaven.test.skip=true -gs `pwd`/configuration/settings.xml clean package'
                  sh 'docker build -f Dockerfile -t $REGISTRY/$DOCKERHUB_NAMESPACE/$APP_NAME:$BRANCH_NAME-${GIT_HASH}-$BUILD_NUMBER .'
                  withCredentials([usernamePassword(passwordVariable : 'DOCKER_PASSWORD' ,usernameVariable : 'DOCKER_USERNAME' ,credentialsId : "$DOCKER_CREDENTIAL_ID" ,)]) {
                      sh 'echo "$DOCKER_PASSWORD" | docker login $REGISTRY -u "$DOCKER_USERNAME" --password-stdin'
                      sh 'docker push  $REGISTRY/$DOCKERHUB_NAMESPACE/$APP_NAME:$BRANCH_NAME-${GIT_HASH}-$BUILD_NUMBER'
                  }
              }
          }
      }

      stage('deploy to dev') {
        steps {
          kubernetesDeploy(configs: 'deploy/dev/**', enableConfigSubstitution: true, kubeConfig: [path: '.kube/config'])
        }
      }
  }
}
