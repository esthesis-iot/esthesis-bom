pipeline {
    agent {
        kubernetes {
            inheritFrom 'cicd-agent'
            yaml '''
              apiVersion: v1
              kind: Pod
              metadata:
                name: esthesis-bom
                namespace: jenkins
              spec:
                priorityClassName: jenkins-low-priority
                securityContext:
                  runAsUser: 0
                  runAsGroup: 0
                  fsGroup: 0
                containers:
                - name: esthesis-bom-builder
                  image: eddevopsd2/ubuntu-dind:docker24-mvn3.9.6-jdk21-node18.16-go1.23-buildx-helm
                  volumeMounts:
                  - name: maven
                    mountPath: /root/.m2/
                    subPath: esthesis
                  tty: true
                  securityContext:
                    privileged: true
                    runAsUser: 0
                  resources:
                    requests:
                      cpu: "500m"
                      memory: "1Gi"
                    limits:
                      cpu: "1"
                      memory: "2Gi"
                imagePullSecrets:
                - name: regcred
                volumes:
                - name: maven
                  persistentVolumeClaim:
                    claimName: maven-nfs-pvc
            '''
            workspaceVolume persistentVolumeClaimWorkspaceVolume(claimName: 'workspace-nfs-pvc', readOnly: false)
        }
    }
    options {
        disableConcurrentBuilds()
        buildDiscarder(logRotator(numToKeepStr: '10'))
        timeout(time: 3, unit: 'HOURS')
    }
    stages {
        stage('Build') {
            steps {
                container (name: 'esthesis-bom-builder') {
                    sh 'mvn clean install'
                }
            }
        }
    }
    post {
        changed {
            emailext subject: '$DEFAULT_SUBJECT',
                body: '$DEFAULT_CONTENT',
                to: '12133724.eurodynlu.onmicrosoft.com@emea.teams.ms'
        }
    }
}