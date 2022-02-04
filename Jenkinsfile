node {
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
        sh 'docker push asirsikar319/${serviceName}:${imageVersion}'
  }
  if('${deploymentType}' == 'New'){
    stage('Deploy') {
    withKubeConfig([credentialsId: 'asirsikar319_kubernates',
                    serverUrl: 'https://kubernetes.docker.internal:6443',
                    contextName: 'docker-desktop',
                    clusterName: 'docker-desktop',
                    namespace: '${namespace}'
                    ]) {
      sh 'kubectl apply -f deployment.yaml'
      sh 'kubectl create configmap ${serviceName} --from-file=etc/config/ -n ${namespace}'      
    }
   }
  }else{
    stage('Deploy') {
    withKubeConfig([credentialsId: 'asirsikar319_kubernates',
                    serverUrl: 'https://kubernetes.docker.internal:6443',
                    contextName: 'docker-desktop',
                    clusterName: 'docker-desktop',
                    namespace: '${namespace}'
                    ]) {
      sh 'kubectl delete -f deployment.yaml'
      sh 'kubectl delete configmap ${serviceName} -n ${namespace}'
      sh 'kubectl apply -f deployment.yaml'
      sh 'kubectl create configmap ${serviceName} --from-file=etc/config/ -n ${namespace}'      
    }
   }
  }
}
