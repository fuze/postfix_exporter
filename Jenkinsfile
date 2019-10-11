@Library('infra-jenkins@master') _

stage('checkout') {
  node('infra') {
    try {
      sh '''
        git clone https://github.com/kumina/postfix_exporter.git
      '''
    } catch (e) {
      echo e.getMessage()
      currentBuild.result = 'FAILURE'
    }
  }
}

stage('compile') {
  if (currentBuild.result != 'FAILURE') {
    node('infra') {
      try {
        docker.image('centos:7').inside('-u 0:0') {
          sh '''
            yum install -y epel-release
            yum install -y systemd-devel
            yum install -y golang
            mkdir -p /go/src/postfix_exporter
            mv * /go/src/postfix_exporter
            export GOPATH=/go
            pushd .
            cd /go/src/postfix_exporter
            go get
            go build
            popd
            mv /go/bin/postfix_exporter .
          '''
        }
        if (env.BRANCH_NAME != 'master') {
          archiveArtifacts artifacts: 'postfix_exporter', fingerprint: true
        }
      } catch (e) {
        echo e.getMessage()
        currentBuild.result = 'FAILURE'
      }
    }
  }
}

stage('upload') {
  if (currentBuild.result != 'FAILURE') {
    if (env.BRANCH_NAME == 'master') {
      node('infra') {
        try {
          sendToNexus("infra-postfix-exporter", "postfix_exporter", "go/bin")
        } catch (e) {
          echo e.getMessage()
          currentBuild.result = 'FAILURE'
        }
      }
    } else {
      echo "Skipping, not master branch"
    }
  }
}

stage('cleanup') {
  node('infra') {
    step([$class: 'WsCleanup'])
  }
}
