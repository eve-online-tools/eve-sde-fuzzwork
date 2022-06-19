pipeline {
  agent {

        kubernetes {
yaml """
apiVersion: v1
kind: Pod
metadata:
  name: jenkins-slave
spec:
  containers:
  - name: jnlp
    image: jenkins/inbound-agent:4.10-3-jdk11
    imagePullPolicy: IfNotPresent
    tty: true
  - name: tools
    image: ghcr.io/eve-online-tools/jenkins-tools:0.2
    imagePullPolicy: IfNotPresent
    command:
    - cat
    tty: true
  - name: dind
    image: ghcr.io/eve-online-tools/dind-buildx:0.3
    imagePullPolicy: IfNotPresent
    securityContext:
      privileged: true
    tty: true
    readinessProbe:
      exec:
        command: ["docker", "info"]
      initialDelaySeconds: 10
      failureThreshold: 6
    volumeMounts:
      - name: jenkins-docker-cfg
        mountPath: /root/.docker/config.json
        subPath: config.json
  volumes:
    - name: jenkins-docker-cfg
      projected:
        sources:
        - secret:
            name: externalregistry
            items:
              - key: .dockerconfigjson
                path: config.json
  restartPolicy: Never
  nodeSelector:
    kubernetes.io/arch: amd64
  imagePullSecrets:
    - name: externalregistry
"""
 idleMinutes 10
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
              //sh "docker buildx build --platform linux/amd64,linux/arm64/v8 -f `pwd`/Dockerfile -t $TARGET_REGISTRY/eve-mariadb-sde:`cat version.txt` --push `pwd`"
          }
        }
      }

      stage('deploy') {
          steps {
            container('tools') {
                withKubeConfig([credentialsId: "k8s-credentials", serverUrl: "https://rancher.rwcloud.org/k8s/clusters/local", restrictKubeConfigAccess: "true" ]) {
                    sh 'kubectl -n $NAMESPACE get pods'
                    //sh 'helm -n $NAMESPACE upgrade -i eve-sde-db `pwd`/helm/eve-sde-db --set image.tag=`cat version.txt` --wait'
                    sh 'helm -n $NAMESPACE upgrade -i eve-sde-db `pwd`/helm/eve-sde-db --set image.tag=20220524 --wait'
                }
            }
          }
      }
  }
}
