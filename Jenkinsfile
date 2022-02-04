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
  stage('Deploy') {
    withKubeConfig([credentialsId: 'asirsikar319_kubernates',
                    serverUrl: 'https://kubernetes.docker.internal:6443',
                    contextName: 'docker-desktop',
                    clusterName: 'docker-desktop',
                    namespace: '${namespace}'
                    ]) {
      when {
        expression {
           'service/${serviceName}' ==  sh(returnStdout: true, script: 'kubectl get svc ${serviceName} -n ${namespace} -o=name').trim()
        }
      }
      steps {
        sh 'kubectl delete -f deployment.yaml'
        sh 'kubectl delete configmap ${serviceName} -n ${namespace}'
        sh 'kubectl apply -f deployment.yaml'
        sh 'kubectl create configmap ${serviceName} --from-file=etc/config/ -n ${namespace}'
      }
      sh 'kubectl apply -f deployment.yaml'
      //sh 'kubectl expose deployment ${serviceName} --type=NodePort --port=8080 --target-port=8080 -n ${namespace}'
      sh 'kubectl create configmap ${serviceName} --from-file=etc/config/ -n ${namespace}'
      
    }
    
  }
}
