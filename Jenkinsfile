def remote = [name: 'k8 master', host: '172.31.13.36', user: 'ubuntu', password: 'apple@2021', allowAnyHosts: true]
pipeline {
  environment {
    registry = "gnishanth444/productsonkubernetes"
    registryCredential = 'docker-creds'
    dockerImage = ''
  }
  agent any
  stages {
    stage('Cloning Git') {
      steps {
        git 'https://github.com/gnishanth4/ProductsStoreOnKubernetes.git'
      }
    }
    stage('Building image') {
      steps{
        
        script {
          dir('mvcapp') {
          dockerImage = docker.build registry + ":$BUILD_NUMBER"
          }
        }
      }
    }

    stage('Deploy Image to Hub') {
      steps{
        script {
          withDockerRegistry([ credentialsId: registryCredential,url: ""] ) {
            
            sh 'docker push gnishanth444/productsonkubernetes":$BUILD_NUMBER"'
          }
        }
      }
    }
    stage('Remove Unused docker image') {
      steps{
        sh "docker rmi $registry:$BUILD_NUMBER"
      }
    }

    stage('Put deployment.yml onto k8smaster'){

      steps{
         
         sshPut remote: remote, from: 'kubernetes-deployment.yaml', into: '.'

      }  
    }
    stage('Deploy ASP_Dot_App') {
      steps{
        script{
          try{
           
           sshCommand remote: remote, command: "sudo kubectl apply -f kubernetes-deployment.yaml"
          }
          catch(error){
           
            sshCommand remote: remote, command: "sudo kubectl create -f kubernetes-deployment.yaml"
          }
        }
      }
     }
    stage('Run Kubectl Command'){
      steps{
         
         sshCommand remote: remote, command: "sudo kubectl get pods"
      }
    }
  }
}
