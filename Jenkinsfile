def _message

pipeline {
  agent any
  options {
    disableConcurrentBuilds()
    timeout(time: 30, unit: 'MINUTES')
    buildDiscarder(logRotator(numToKeepStr: '30'))
  }

  parameters {
    string(name: '_git_repo', defaultValue: 'git@github.com:KuzmenkoAlexey/real-world-app-angular.git')
    string(name: '_git_branch', defaultValue: 'main' )
  }

  stages {
    stage('Git checkout') {
        steps {
            script {
              echo "Git checkout 1"
            }
            checkout([
              $class: 'GitSCM',
              userRemoteConfigs: [[credentialsId: "git", url: "${_git_repo}"]],
              branches: [[name: "${_git_branch}"]]
            ])
            script {
              echo "Git checkout 2"
              _git_commit = sh(returnStdout: true, script: "git log -n 1 --pretty=format:'%h'").trim()
              echo "Git Commit: ${_git_commit}"
            }
        }
    }

    node {
        stage('Deploy') {
            steps {
                withKubeConfig([
                  credentialsId: 'ede8d86c-dbd4-4837-aa43-24b4fe852bd7',
                  serverUrl: 'https://34.121.97.129',
                  clusterName: 'gke_data-buckeye-288515_us-central1-a_kuzmenko-cluster',
                  namespace: 'kuzmenko-onboarding'
                ]) {
                  sh 'kubectl get namespaces'
                }
            }
        }
    }
  }

  post {
   success {
      script {
        echo "success!!!"
      }
    }
  }

}
