pipeline {
  environment {
    GET_CONTENT = "true"
    NODE_ENV = "production"
    HOME = "/tmp"
    TZ = "UTC"
  }

  agent {
    label 'docker&&linux'
  }

  options {
    timeout(time: 60, unit: 'MINUTES')
    ansiColor('xterm')
    buildDiscarder logRotator(artifactDaysToKeepStr: '', artifactNumToKeepStr: '', daysToKeepStr: '5', numToKeepStr: '5')
  }

  stages {
    stage('Check for typos') {
      steps {
        sh '''
          curl -qsL https://github.com/crate-ci/typos/releases/download/v1.5.0/typos-v1.5.0-x86_64-unknown-linux-musl.tar.gz | tar xvzf - ./typos
          curl -qsL https://github.com/halkeye/typos-json-to-checkstyle/releases/download/v0.1.1/typos-checkstyle-v0.1.1-x86_64 > typos-checkstyle && chmod 0755 typos-checkstyle
          ./typos --format json | ./typos-checkstyle - > checkstyle.xml || true
        '''
        recordIssues(tools: [checkStyle(id: 'typos', name: 'Typos', pattern: 'checkstyle.xml')])
      }
    }

    stage('NPM Install') {
      agent {
        docker {
          image 'node:14.17'
          reuseNode true
        }
      }
      steps {
        sh('yarn install')
      }
    }

    stage('Build Production') {
      agent {
        docker {
          image 'node:14.17'
          reuseNode true
        }
      }
      steps {
        sh('yarn build')
      }
    }

    stage('Check build') {
      agent {
        docker {
          image 'node:14.17'
          reuseNode true
        }
      }
      steps {
        sh 'test -e ./plugins/plugin-site/public/index.html || exit 1'
      }
    }

    stage('Lint and Test') {
      agent {
        docker {
          image 'node:14.17'
          reuseNode true
        }
      }
      steps {
        sh('yarn lint')
        sh('yarn test')
      }
    }
  }
}
