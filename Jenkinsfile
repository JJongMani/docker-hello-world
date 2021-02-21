podTemplate(label: 'docker-build', 
  containers: [
    containerTemplate(
      name: 'docker',
      image: 'docker',
      command: 'cat',
      ttyEnabled: true
    ),
    containerTemplate(
      name: 'argo',
      image: 'argoproj/argo-cd-ci-builder:latest',
      command: 'cat',
      ttyEnabled: true
    ),
  ],
  volumes: [ 
    hostPathVolume(mountPath: '/var/run/docker.sock', hostPath: '/var/run/docker.sock'), 
  ]
) {
    node('docker-build') {
        def dockerHubCred = "dockerhub_cred"
        def appImage
        
        stage('Checkout'){
            container('argo'){
                checkout scm
            }
        }
        
        stage('Build'){
            container('docker'){
                script {
                    appImage = docker.build("arm7tdmi/node-hello-world")
                }
            }
        }
        
        stage('Test'){
            container('docker'){
                script {
                    appImage.inside {
                        sh 'npm install'
                        sh 'npm test'
                    }
                }
            }
        }

        stage('Push'){
            container('docker'){
                script {
                    docker.withRegistry('https://registry.hub.docker.com', dockerHubCred){
                        appImage.push("${env.BUILD_NUMBER}")
                        appImage.push("latest")
                    }
                }
            }
        }

        def gitHubCred = "my_github_cred"
        stage('Deploy'){
            container('argo'){
                withCredentials([usernamePassword(credentialsId: gitHubCred, usernameVariable: 'USERNAME', passwordVariable: 'PASSWD')]) {
                    script {
                        env.ENCODED_PASSWD=URLEncoder.encode(PASSWD, "UTF-8")
                        env.GIT_URL='github.com/cure4itches/docker-hello-world-deployment'
                    }
                    sh 'git clone https://${USERNAME}:${ENCODED_PASSWD}@${GIT_URL}'
                    dir("docker-hello-world-deployment"){
                        sh 'cd env/dev'
                        sh 'kustomize edit set image arm7tdmi/node-hello-world:${BUILD_NUMBER}'
                        sh 'git commit -a -m "updated the image tag'
                        sh 'cd -'
                        sh 'git push'
                    }
                }
            }
        }
    }
    
}