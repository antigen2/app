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
        withKubeConfig([credentialsId: 'k8s',
                caCertificate: 'LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUMvakNDQWVhZ0F3SUJBZ0lCQURBTkJna3Foa2lHOXcwQkFRc0ZBREFWTVJNd0VRWURWUVFERXdwcmRXSmwKY201bGRHVnpNQjRYRFRJek1EY3lPREUzTXpneU9Gb1hEVE16TURjeU5URTNNemd5T0Zvd0ZURVRNQkVHQTFVRQpBeE1LYTNWaVpYSnVaWFJsY3pDQ0FTSXdEUVlKS29aSWh2Y05BUUVCQlFBRGdnRVBBRENDQVFvQ2dnRUJBTitjCkRpSytlMGpyTlp4ek5NSU5FUGZQdldVMEUySWQvNTEzUUU2dTViVWVwSTNqRGhNeDBLNWFpa3pCanIxV1d4eC8KRjFiQkhEdUg4WGU5OHBGV3hhdER5LzU4UjhYZmFJYkprYVg5L3M4SXJpWVBVL2RWQWUvZzd1SVdMR1RKczZmMgpKaXR5WjR0RDUxaHZZM1RFanpIT0tpdUptcVBud01ScEZDMDVOcVhPS2p6T3Z2OVQ5N0o1enJNRXNmaUplMlJ3CmVDYXdCM28ybXhHQVlEWUV5dlJJdm56SWpmK2lWVmdCQWZsbE5leldPazQ1MjZteHc2Yi8rNW1SNjZiWjlYc2sKTFdJancvVUJ3RGtvUEU3VTMxamhUeXF5eFBlVlY3M0F4YzRvcVlKZUM4bnBMRElHVjdMWXo0Z2JGL2J0WmliTApaSFlueUdjbmF2U1Y3R1pmTXVVQ0F3RUFBYU5aTUZjd0RnWURWUjBQQVFIL0JBUURBZ0trTUE4R0ExVWRFd0VCCi93UUZNQU1CQWY4d0hRWURWUjBPQkJZRUZKWnBkSmc0bGIyNm5pKzFuSGhvL3pWKzFGaXlNQlVHQTFVZEVRUU8KTUF5Q0NtdDFZbVZ5Ym1WMFpYTXdEUVlKS29aSWh2Y05BUUVMQlFBRGdnRUJBRTdzQVl0VG44TEZnanNmQjJJUwpsYmdESkdSWUFGSTdrTjZMUnpCcE1IY0FKM040aWwvQnNXSzJTQ0lzRm9mQ0dpOTZ5ZmVjd2wrMEdrR2ZXRzVLCnh5Yk1nWlFxZ3QweGptUy9GNlMyNjBleFZ3M1JwZjVycUVWSFVJS2ZWek1EZ3h0VmdOZ1UwdFhlSTAvVHlRNEoKaHMxcG1PZ3ZqcWRyanp5bWt1S29IendqTExLeWZxS1B3SnJCektES2ppQ1VmcllHVTgzbHF4dDlaa20ydDlZcwpYWmtrWVBiWWxIYlJxN2c0QWQxOWcwL3ZNY0RSdVQ4Q3RrRmV2Q29ycDQ3enV2dTNDSTFEOHA1dStrSVV6Vm1qCkQ1Q1VqbWN3L2tiSWhEUXJOdEZyVitMeStLNXNKTjJ5cEcwL2FKbWVUa2R6THpwanR4N1paeFlYcVhhd2RSakIKdTdRPQotLS0tLUVORCBDRVJUSUZJQ0FURS0tLS0tCg==',
                serverUrl: 'https://158.160.11.73:6443',
                contextName: 'kubernetes-admin@cluster.local',
                clusterName: 'cluster.local',
                namespace: 'stage'
                ]) {
          sh 'kubectl apply -f app-deploy.yml'
        }
      }
    }




  }    
}    
