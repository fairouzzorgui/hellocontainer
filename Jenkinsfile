podTemplate(label: 'buildpod',
    volumes: [
        hostPathVolume(hostPath: '/etc/docker/certs.d', mountPath: '/etc/docker/certs.d'),
        hostPathVolume(hostPath: '/var/run/docker.sock', mountPath: '/var/run/docker.sock'),
        secretVolume(secretName: 'defregistrykey', mountPath: '/var/run/secrets/registry-account'),
        configMapVolume(configMapName: 'def-registry-config', mountPath: '/var/run/configs/registry-config')
    ],
    imagePullSecrets:['defregistrykey'],
    containers: [
        containerTemplate(name: 'node', image: 'node:6.12.0-alpine', command: 'cat', ttyEnabled: true),
        containerTemplate(name: 'docker', image: 'mycluster.icp:8500/default/docker:latest', command: 'cat', ttyEnabled: true, imagePullSecrets:['defregistrykey'],alwaysPullImage: true),
    ]) {

    node('buildpod') {
        checkout scm
        container('node') {
            stage('Build') {
                sh 'npm install'
            }
            stage('Test'){
                sh 'npm test'
            }
        }
        container('docker') {
            stage('Build Docker Image') {
                sh """
                #!/bin/bash
                NAMESPACE='default'
                REGISTRY='mycluster.icp:8500'

                docker build -t \${REGISTRY}/\${NAMESPACE}/hello-container:${env.BUILD_NUMBER} .
                """
            } 
            stage('Push Docker Image to Registry') {
                sh """
                #!/bin/bash
                NAMESPACE='default'
                REGISTRY='mycluster.icp:8500'

                set +x
                DOCKER_USER='admin'
                DOCKER_PASSWORD='admin'
                docker login -u=\${DOCKER_USER} -p=\${DOCKER_PASSWORD} \${REGISTRY}
                set -x

                docker push \${REGISTRY}/\${NAMESPACE}/hello-container:${env.BUILD_NUMBER}
                """
            }
        }

    }
            node('master') {
            checkout scm
            stage('Deploy on kubernetes') {
               // podTemplate(label: 'hellocontainer', inheritFrom: 'deployment.yml')
               // {
                ///testing
                //}
            //    kubernetesDeploy(kubeconfigId: 'confkube',configs: 'deployment.yml',enableConfigSubstitution: true)
                  withKubeConfig([credentialsId: 'confkube', serverUrl: 'https://192.168.0.150']) {
          //          steps{
                        sh 'kubectl apply -f deployment.yml'
                  }
                    
        }
        //}     
        }

}
