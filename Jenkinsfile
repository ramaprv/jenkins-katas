pipeline {
  agent any
  stages {
    stage("clone down") {
      agent { label "swarm" }
      steps {
        stash excludes: ".git", name: "code"
      }
    }
    stage('Parallel execution') {
      parallel {
        stage('Say hello') {
          steps {
            sh 'echo "hello world"'
          }
        }

        stage('build app') {
          agent {
            docker {
              image 'gradle:6.9.3-jdk11'
            }
          }
          options {
            skipDefaultCheckout true
          }
          steps {
            unstash "code"
            sh 'ci/build-app.sh'
            archiveArtifacts 'app/build/libs/'
          }
        }
        stage ("test app") {
          agent {
            docker {
              image 'gradle:6.9.3-jdk11'
            }
          }
          options {
            skipDefaultCheckout true
          }
          steps {
            unstash "code"
            sh "ci/unit-test-app.sh"
            junit 'app/build/test-results/test/TEST-*.xml'
          }
        }
      }
    }
  }
  post {
    cleanup {
      // cleanWs()
      deleteDir()
    }
  }
}
