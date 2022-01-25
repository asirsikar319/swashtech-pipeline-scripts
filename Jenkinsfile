node {
  parameters{
      choice(name: "selectRepository", choices: ['https://github.com/asirsikar319/swashtech-service.git', 'https://github.com/asirsikar319/swashtech-quickcare-service.git', 'https://github.com/asirsikar319/swashtech-rushmgmt-service.git'])
  }
  stage("Git Clone"){
      git credentialsId: 'asirsikar319_github', url: '${selectRepository}'
  }
  stage("Build"){
      sh 'mvn clean install'
  }
  stage("Building Image"){
        sh 'mvn clean package docker:build'
  }
  withCredentials([string(credentialsId: 'dockerhub_pwd', variable: 'PASSWORD')]) {
        sh 'docker login -u asirsikar319 -p $PASSWORD'
  }
  stage("Pushing Image"){
        sh 'docker push asirsikar319/swashtech-service:latest'
  }
  stage('Deploy') {
    withKubeConfig([credentialsId: 'asirsikar319_kubernates',
                    serverUrl: 'https://kubernetes.docker.internal:6443',
                    contextName: 'docker-desktop',
                    clusterName: 'docker-desktop',
                    namespace: 'swashtech'
                    ]) {
      sh 'kubectl apply -f deployment.yaml'
      sh 'kubectl expose deployment swashtech-service --type=NodePort --port=8080 --target-port=8080 -n swashtech'
      sh 'kubectl create configmap swashtech-service --from-file=etc/config/ -n swashtech'
    }
  }
}
