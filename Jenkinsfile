pipeline {
  agent any
  stages {
    stage('Clone Down') {
      options {
        skipDefaultCheckout(true)
      }
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

        stage('Build App') {
          agent {
            docker {
              image 'gradle:jdk11'
            }

          }
          when {
            beforeAgent true
            branch 'master'
          }
          steps {
            sh 'ci/build-app.sh'
            stash 'code'
            sh 'ls'
            deleteDir()
            sh 'ls'
          }
        }

        stage('Test App') {
          agent {
            docker {
              image 'gradle:jdk11'
            }

          }
          post {
            always {
              deleteDir()
            }

          }
          steps {
            unstash 'code'
            sh 'ci/unit-test-app.sh'
            junit 'app/build/test-results/test/TEST-*.xml'
          }
        }

      }
    }

    stage('push docker app') {
      environment {
        DOCKERCREDS = credentials('docker_login')
      }
      steps {
        unstash 'code'
        sh 'ci/build-docker.sh'
        sh 'echo "$DOCKERCREDS_PSW" | docker login -u "$DOCKERCREDS_USR" --password-stdin'
        sh 'ci/push-docker.sh'
      }
    }

    stage('Component Test') {
      when {
        not {
          branch 'dev/*'
        }

      }
      steps {
        sh 'ci/component-test.sh'
        sh 'echo done'
      }
    }

  }
  environment {
    docker_username = 'euronames'
  }
}