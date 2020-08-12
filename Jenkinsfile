pipeline {
  agent any
  stages {
    stage('__clone down__') {
      options {
        skipDefaultCheckout(true)
      }
      steps {
        stash(excludes: '.git', name: 'code')
      }
    }

    stage('Parallel execution') {
      options {
        skipDefaultCheckout(true)
      }
      parallel {
        stage('Say Hello') {
          options {
            skipDefaultCheckout(true)
          }
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
          options {
            skipDefaultCheckout(true)
          }
          steps {
            unstash 'code'
            sh 'ci/build-app.sh'
            archiveArtifacts 'app/build/libs/'
            stash(excludes: '.git', name: 'buildfiles')
            sh 'ls -a'
            deleteDir()
            sh 'ls -a'
          }
        }

        stage('test app') {
          agent {
            docker {
              image 'gradle:jdk11'
            }

          }
          options {
            skipDefaultCheckout(true)
          }
          steps {
            sh 'ci/unit-test-app.sh'
            junit 'app/build/test-results/test/TEST-*.xml'
          }
        }

      }
    }

    stage('docker_push') {
      when {
        branch 'master'
      }
      environment {
        docker_username = 'euronames'
        DOCKERCREDS = credentials('docker_login')
      }
      options {
        skipDefaultCheckout(true)
      }
      steps {
        unstash 'buildfiles'
        sh 'ci/build-docker.sh'
        sh 'echo "$DOCKERCREDS_PSW" | docker login -u "$DOCKERCREDS_USR" --password-stdin'
        sh 'ci/push-docker.sh'
      }
    }

    stage('component test') {
      agent {
        docker {
          image 'gradle:jdk11'
        }

      }
      when {
        not {
          branch 'dev/'
        }

      }
      options {
        skipDefaultCheckout(true)
      }
      steps {
        unstash 'code'
        sh 'ci/component-test.sh'
      }
    }

  }
}