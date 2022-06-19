pipeline {
  agent {
        kubernetes {
            yamlFile 'KubernetesPod.yaml'
        }
  }
    
  environment {
    TARGET_REGISTRY = "ghcr.io/eve-online-tools"
    NAMESPACE = "eve-dev"
  }

  stages {
    stage('extract version') {
      steps {
        container('dind') {
            sh "wget https://www.fuzzwork.co.uk/dump/mysql-latest.tar.bz2 -O mysql.tar.bz2"
            sh "(tar tfj mysql.tar.bz2 | sed 's/sde-//g' | sed 's/-.*//g') > version.txt"
        }
      }
    }

    stage('create & push docker-image') {
        steps {        
          container('dind') {
              sh "docker buildx create --use"
              sh "docker buildx build --platform linux/amd64,linux/arm64/v8 -f `pwd`/Dockerfile -t $TARGET_REGISTRY/eve-mariadb-sde:`cat version.txt` --push `pwd`"
          }
        }
      }

      stage('deploy') {
          steps {
            container('tools') {

                withCredentials([string(credentialsId: 'k8s-server-url', variable: 'SERVER_URL')]) {
                    withKubeConfig([credentialsId: "k8s-credentials", serverUrl: "$SERVER_URL"]) {
                        sh 'kubectl -n $NAMESPACE get pods'
                        sh 'helm -n $NAMESPACE upgrade -i eve-sde-db `pwd`/helm/eve-sde-db --set image.tag=`cat version.txt` --wait'
                    }
                }
            }
          }
      }
  }
}
