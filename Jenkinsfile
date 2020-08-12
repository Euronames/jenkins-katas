pipeline {
  agent any
  stages {
    stage('__clone down__') {
      steps {
        stash(excludes: '.git', name: 'code')
      }
    }

    stage('Parallel execution') {
      parallel {
        stage('Say Hello') {
          steps {
            sh 'echo "hello world"'
          }
        }

        stage('build app') {
          agent {
            docker {
              image 'gradle:jdk11'
            }

          }
          steps {
            sh 'ci/build-app.sh'
            archiveArtifacts 'app/build/libs/'
            sh 'ls -a'
            deleteDir()
            sh 'ls -a'
            skipDefaultCheckout true
            stash(excludes: '.git', name: 'code')
          }
        }

      }
    }

    stage('test app') {
      agent {
        docker {
          image 'gradle:jdk11'
        }

      }
      steps {
        sh 'ci/unit-test-app.sh'
        junit 'app/build/test-results/test/TEST-*.xml'
      }
    }

    stage('docker_push') {
      steps {
        unstash 'code'
        sh 'ci/build-docker.sh'
        sh 'echo "$DOCKERCREDS_PSW" | docker login -u "$DOCKERCREDS_USR" --password-stdin'
        sh 'ci/push-docker.sh'
      }
    }

  }
  environment {
    DOCKERCREDS = credentials('docker_login')
  }
}