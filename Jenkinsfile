podTemplate(label: 'buildpod',
    volumes: [
        hostPathVolume(hostPath: '/etc/docker/certs.d', mountPath: '/etc/docker/certs.d'),
        hostPathVolume(hostPath: '/var/run/docker.sock', mountPath: '/var/run/docker.sock'),
        secretVolume(secretName: 'defregistrykey', mountPath: '/var/run/secrets/registry-account'),
        configMapVolume(configMapName: 'def-registry-config', mountPath: '/var/run/configs/registry-config')
    ],
    imagePullSecrets:['defregistrykey'],
    containers: [
        containerTemplate(name: 'docker', image: 'mycluster.icp:8500/default/docker:latest', command: 'cat', ttyEnabled: true, imagePullSecrets:['defregistrykey'],alwaysPullImage: true),
        containerTemplate(name: 'helm', image: 'mycluster.icp:8500/default/k8s-helm:latest', command: 'cat', ttyEnabled: true,imagePullSecrets:['defregistrykey'],alwaysPullImage: true)
  ]) {

    node('buildpod') {
        checkout scm
        container('docker') {
            stage('Build Docker Image') {
                sh """
                #!/bin/bash
                NAMESPACE=`default`
                REGISTRY=`mycluster.icp:8500`

                docker build -t \${REGISTRY}/\${NAMESPACE}/hello-container:${env.BUILD_NUMBER} .
                """
            } 
            stage('Push Docker Image to Registry') {
                sh """
                #!/bin/bash
                NAMESPACE=`default`
                REGISTRY=`mycluster.icp:8500`

                set +x
                DOCKER_USER=`admin`
                DOCKER_PASSWORD=`admin`
                docker login -u=\${DOCKER_USER} -p=\${DOCKER_PASSWORD} \${REGISTRY}
                set -x

                docker push \${REGISTRY}/\${NAMESPACE}/hello-container:${env.BUILD_NUMBER}
                """
            }
        }

        container('helm') {
            stage('Deploy new helm release') {
                sh """
                #!/bin/bash
                set +e
                NAMESPACE=`default`
                REGISTRY=`mycluster.icp:8500`
                CHARTNAME=`helm list --deployed --short hello-container`

                helm list \${CHARTNAME}

                if [ \${?} -ne "0" ]; then
                    # No chart release to update
                    echo 'No chart release to update'
                    exit 1
                fi

                # Update Release 
                helm upgrade hello-container ./hellocontainer-chart/ --set image.repository=\${REGISTRY}/\${NAMESPACE}/hello-container --set image.tag=${env.BUILD_NUMBER}
                """
            }
        }


    }
}
