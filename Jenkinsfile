pipeline {
  environment {
    dockerimagename = "antigen2/app"
    dockerImage = ""
    
  }
  agent any
  stages {
    stage('Checkout Source') {
      steps {
        git branch: 'main', url: 'https://github.com/antigen2/app.git'
      }
    }
    stage('Checkout tag') {
      steps{
        script {
          sh 'git fetch'
          gitTag=sh(returnStdout:  true, script: "git tag --sort=-creatordate | head -n 1").trim()
          echo "gitTag output: ${gitTag}"
        }
      }
    }
    stage('Build image') {
      steps{
        script {
          dockerImage = docker.build dockerimagename
        }
      }
    }
    stage('Pushing Image:tags') {
      environment {
               registryCredential = 'dockerhub-credentials'
           }
      steps{
        script {
          docker.withRegistry( 'https://index.docker.io/v1/', registryCredential ) {
            dockerImage.push("${gitTag}")
          }
        }
      }
    }
    stage('sed env') {
      environment {
              envTag = ("${gitTag}")
           }    
      steps{
        script {
          sh "sed -i \'18,22 s/gitTag/\'$envTag\'/g\' app-deploy.yml"
          sh 'cat app-deploy.yml'
        }
      }
    }

    stage('Apply Kubernetes files') {
      steps {
        withKubeConfig([credentialsId: 'k8s', serverUrl: 'https://158.160.11.73:6443']) {
          sh '$(which kubectl) apply -f app-deploy.yml'
        }
      }
    }


  }    
}    
