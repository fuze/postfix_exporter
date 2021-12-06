@Library('infra-jenkins@master') _

/**
 * Job Pipeline
 */
node('ec2_vam') {
  try {
    stageCheckout()
    stageCompile()
    if (env.BRANCH_NAME == 'master') {
      sendToNexus()
    }
  } catch (e) {
    currentBuild.result = "FAILURE"
    throw e
  } finally {
    stageCleanup()
  }
}


def stageCheckout() {
  stage('Checkout Repository') {
    checkout scm
  }
}

def stageCompile() {
  stage('compile') {
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
  }
}

def sendToNexus() {
  stage("Send to Nexus") {
    sendToNexus("infra-postfix-exporter", "postfix_exporter", ".")
  }
}

def stageCleanup() {
  stage('Cleanup') {
    cleanWs()
    dir("${env.WORKSPACE}@tmp") {
      deleteDir()
    }
    dir("${env.WORKSPACE}@script") {
      deleteDir()
    }
    dir("${env.WORKSPACE}@script@tmp") {
      deleteDir()
    }
  }
}
