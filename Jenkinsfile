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
  - name: docker
    image: docker:latest
    command:
      - cat
    tty: true
    volumeMounts:
    - mountPath: /var/run/docker.sock
      name: docker-sock
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
    - name: docker-sock
      hostPath:
        path: /var/run/docker.sock
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
    stage('create & push docker-image') {
        steps {        
          container('docker') {
              sh "docker build -f `pwd`/Dockerfile -t $TARGET_REGISTRY/eve-mariadb-sde:`date +%Y%m%d` `pwd`"
          }
        }
      }
  }
}
