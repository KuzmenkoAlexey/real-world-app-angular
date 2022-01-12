def _message

pipeline {
  agent {
      kubernetes {
      defaultContainer 'jnlp'
      yaml """
apiVersion: v1
kind: Pod
metadata:
labels:
  component: ci
spec:
  serviceAccountName: jenkins-robot
  containers:
  - name: docker
    image: docker:20.10.12
    command:
    - cat
    tty: true
    volumeMounts:
    - mountPath: /var/run/docker.sock
      name: docker-sock
  - name: kctl
    image: alpine/k8s:1.21.2
    command:
    - cat
    tty: true
  volumes:
    - name: docker-sock
      hostPath:
        path: /var/run/docker.sock
"""
    }
  }
  options {
    disableConcurrentBuilds()
    timeout(time: 30, unit: 'MINUTES')
    buildDiscarder(logRotator(numToKeepStr: '30'))
  }

  parameters {
    string(name: '_git_repo', defaultValue: 'https://github.com/KuzmenkoAlexey/real-world-app-angular.git')
    string(name: '_git_branch', defaultValue: 'main' )
    string(name: '_gcp_repo', defaultValue: 'gcr.io/data-buckeye-288515/angular-frontendapp')
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

    stage('Build') {
        steps {
            container('docker') {
                script {
                    _build_args = """\
                      --network=host \
                    """
                    sh """
                        #!/bin/bash
                        echo "deploy stage";
                        curl -o /tmp/google-cloud-sdk.tar.gz https://dl.google.com/dl/cloudsdk/channels/rapid/downloads/google-cloud-sdk-225.0.0-linux-x86_64.tar.gz;
                        tar -xvf /tmp/google-cloud-sdk.tar.gz -C /tmp/;
                        /tmp/google-cloud-sdk/install.sh -q;

                                    source /tmp/google-cloud-sdk/path.bash.inc;
                    """
                    withCredentials([file(credentialsId: 'gcp_sa_key', variable: 'GC_KEY')]) {
                        sh("gcloud auth activate-service-account --key-file=${GC_KEY}")
    //             docker.withRegistry("${_gcp_repo}") {
                        def backendImage = docker.build("${_gcp_repo}:${_git_commit}", "${_build_args} .")
                        backendImage.push()
    //             }
                    }
                }
            }
        }
    }

    stage('Deploy') {
        steps {
            container('kctl') {
                withKubeConfig([
                  credentialsId: 'ede8d86c-dbd4-4837-aa43-24b4fe852bd7',
                  serverUrl: 'https://34.121.97.129',
                  clusterName: 'gke_data-buckeye-288515_us-central1-a_kuzmenko-cluster',
                  namespace: 'kuzmenko-onboarding'
                ]) {
                  sh 'kubectl get po -n kuzmenko-onboarding'
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
