pipeline {
    agent {
      label "jenkins-jxcli"
    }
    environment {
        CHARTMUSEUM_CREDS = credentials('jenkins-x-chartmuseum')
    }
    stages {
        stage('CI Build') {
            when {
                branch 'PR-*'
            }
            steps {
              container('jxcli') {
                  dir ('./charts') {
                      checkout scm
                  }

                  dir ('./charts/stable/jenkins') {
                      sh "helm init --client-only"

                      sh "make build"
                      sh "helm template ."
                  }
              }
            }
        }

        stage('Build and Push Release') {
            when {
                branch 'master'
            }
            steps {
              container('jxcli') {
                dir ('/home/jenkins/jenkins-x-platform') {
                    checkout scm
                }

                dir ('./charts/stable/jenkins') {
                    sh "git checkout master"
                    sh "jx step git credentials"
                    sh "make release"
                }
              }
            }
        }
    }
}
