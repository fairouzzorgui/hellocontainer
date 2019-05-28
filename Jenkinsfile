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
        containerTemplate(name: 'kubectl', image: 'lachlanevenson/k8s-kubectl:latest', command: 'cat', ttyEnabled: true),
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
                docker tag \${REGISTRY}/\${NAMESPACE}/hello-container:${env.BUILD_NUMBER} \${REGISTRY}/\${NAMESPACE}/hello-container:latest 

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

                docker push \${REGISTRY}/\${NAMESPACE}/hello-container:latest
                """
            }
        }
        
        container('kubectl'){
            stage('Deploy on kubernetes') {
                  withKubeConfig([credentialsId: 'token',namespace: 'default',caCertificate: 'MIIFmzCCA4OgAwIBAgIJAI83Ny28YcXiMA0GCSqGSIb3DQEBCwUAMGMxCzAJBgNVBAYTAlVTMREwDwYDVQQIDAhOZXcgWW9yazEPMA0GA1UEBwwGQXJtb25rMRowGAYDVQQKDBFJQk0gQ2xvdWQgUHJpdmF0ZTEUMBIGA1UEAwwLd3d3LmlibS5jb20wIBcNMTkwNTE1MTM1MjQ5WhgPMjExOTA0MjExMzUyNDlaMGMxCzAJBgNVBAYTAlVTMREwDwYDVQQIDAhOZXcgWW9yazEPMA0GA1UEBwwGQXJtb25rMRowGAYDVQQKDBFJQk0gQ2xvdWQgUHJpdmF0ZTEUMBIGA1UEAwwLd3d3LmlibS5jb20wggIiMA0GCSqGSIb3DQEBAQUAA4ICDwAwggIKAoICAQDLhgeOE9/+No9BRr2V6O8u4wjKGP8BpShWiy8SPABa2LhPL1u2gyUHb6zFsF3dBS8vlTTXZ8m+sXEuLn+FoQCKBGfnuEECf3bQmeJO737xEh72h8UQTqJ2nMII6iuKw6ATfWUkP6vj7cTGhWM6mJz7h33Ky/GLSKe8q4+a6kTnB8148BA5gKahNiqVeuuLHtQQkxFTgKIROMmyWxoH/ehUDTwR3NOAfxk5OBoDGEkIIuky3lMvi1/vG3uw8bnW+8nFrYUUHBMDSGbUyfEcr0zhPeitdw6RyGAlUWAYfwMkQb56uDXuRMtUQ+yG/wkTHc1efPIOkH4NwcvDGTMP60yEih9Jkle3OJ6uJCdV35TFPy3FwJWd4LThxpeK3zVCTlxOsxiLdbtLoPdJMPypbUoRtdY9D4lpQlheMDDosZO/Dtx1Bwf1JjYulxjqQuVrk/wkMwtNzIIXYxlGj8RnP52ZYqw9oSRJjZhmIV0J5/c/BFNo8nWUJsqJTZAgcYhTwRhfHQX5NVQkJ4dYIJqXDg3NGg6OZBpKEJ90yzark7YmjeGxZN1CmnhgN8PdRXRCXSxAqtjPWsWd+ievTF6fBViRIT7stJeR3yxcHB9fLY+rQ5525x1MGh5Yz5IhCVD6DO0UPVut5szdLcasGKFrMHA4vC4+Eg/jh7P/UtZySblCAwIDAQABo1AwTjAdBgNVHQ4EFgQUuoXzOscZebhwUjXUNAxrXQQX04UwHwYDVR0jBBgwFoAUuoXzOscZebhwUjXUNAxrXQQX04UwDAYDVR0TBAUwAwEB/zANBgkqhkiG9w0BAQsFAAOCAgEAV9YMasNGom7vq+JSR8ZjysnDHIDoNpflSPIwnbmgLVHjnc0yP2IVwqqXganFZiaTyz8/SN8/+OB9749pTIdOL+qkCU+K5EoG1T9A/CpVl3IuBntb80f8gZMy3VC9BMDxBdTgqFenrlE0sjfLNtXncpXzW8bZUO70ZYlUsI1+twVODl8CxEG7IOM0+holdZ1m7hv/xm4pQ2ZJIWKQdK5xqv3j4B9zatKh+ScmPRxRMLBKB7kZx06sdFb0LVshEeNVtNbazM6r2Yt3FHQ+9eQVZy3qAFyCRBNB5BsHZjp9Jx8nwgi3rwUlckgdvggphu/Az4fnkgGY17so47hDAveDRIH6BdUBcnNKTdQr2JRQidWpjv6lx1aq8C2lnPxXn30SceRthUzqd7VTxY/jYMTZj1qDMSGOWk8M5orxGMv8b8VQhfyycpp6IA/753U1zgMndAwjwBjwWJHry8X7wzJB0yrPMsLfpt79+H9CYipmK7hU89DVLx8pphFaZTN/0LeWyEabshPpICZ5mEb6KgMTqW9jnj+oC/+Mdn/urEjhtAKVTF0TbTiJ04E7jRTPXAsPdJ1AJpJP9ACMninyqqpySlo62KZkpDyfl/odN1pMc4gtsY5nSfh3sKUCMJjuzUjOFGi/0if5ERvqNCYQDzV0Aol6P6NncFn9HpuJvoX/TAI=',
                                  contextName: 'mycluster-context', clusterName: 'mycluster', serverUrl: 'https://192.168.0.150:8001']) {
                        sh 'kubectl config view'
                        sh 'kubectl apply -f deployment.yml' 
                  }
        }
        } 

    }
             
        

}
