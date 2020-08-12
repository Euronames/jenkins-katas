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
            unstash 'code'
            sh 'ci/build-app.sh'
            archiveArtifacts 'app/build/libs/'
            stash(excludes: '.git', name: 'buildfiles')
            sh 'ls -a'
            deleteDir()
            sh 'ls -a'
            skipDefaultCheckout true
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
          steps {
            sh 'ci/component-test.sh'
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
      steps {
        unstash 'buildfiles'
        sh 'ci/build-docker.sh'
        sh 'echo "$DOCKERCREDS_PSW" | docker login -u "$DOCKERCREDS_USR" --password-stdin'
        sh 'ci/push-docker.sh'
      }
    }

  }
}