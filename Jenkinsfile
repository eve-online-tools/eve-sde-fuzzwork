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
  - name: kaniko
    image: gcr.io/kaniko-project/executor:debug
    imagePullPolicy: IfNotPresent
    command:
      - /busybox/sh
      - "-c"
    args:
      - /busybox/cat
    tty: true
    volumeMounts:
      - name: jenkins-docker-cfg
        mountPath: /kaniko/.docker
  volumes:
    - name: m2
      persistentVolumeClaim:
        claimName: cache
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
  }

  stages {
    stage('extract version') {
      steps {
        container('dind') {
            sh "curl https://www.fuzzwork.co.uk/dump/mysql-latest.tar.bz2 -o mysql.tar.bz2"
            sh "export VERSION=\$(tar tfj /tmp/mysql.tar.bz2 | sed 's/sde-//g' | sed 's/-.*//g')"
        }
      }
    }

    stage('create & push docker-image') {
        steps {        
          container('dind') {
              sh "docker buildx create --use"
              sh "docker buildx build --platform linux/amd64,linux/arm64/v8 -f `pwd`/Dockerfile -t $TARGET_REGISTRY/eve-mariadb-sde:${VERSION} --push `pwd`"
          }
        }
      }
  }
}
