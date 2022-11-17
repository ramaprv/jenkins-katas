pipeline {
  agent any
  environment {
    docker_username = "ramaverdant"
  }
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
    stage ("push to docker hub") {
      environment {
            DOCKERCREDS = credentials('docker_login') //use the credentials just created in this stage
      }
      steps {
            unstash 'code' //unstash the repository code
            sh "echo $DOCKERCREDS_USR"
            sh "echo $DOCKERCREDS_PSW"
            sh "echo $DOCKERCREDS"
            sh 'ci/build-docker.sh'
            sh 'echo "$DOCKERCREDS_PSW" | docker login -u "$DOCKERCREDS_USR" --password-stdin' //login to docker hub with the credentials above
            sh 'ci/push-docker.sh'
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
