@Library('infra-jenkins@master') _


node('ec2_vam') {
  try {
    stageCheckout()
    stageCompile()
    if (env.BRANCH_NAME == 'master') {
      sendToNexus()
    } else {
      stageArchiveArtifacts()
    }
  } catch (e) {
    echo e.getMessage()
    currentBuild.result = "FAILURE"
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

def stageArchiveArtifacts() {
  stage("Archive Artifacts") {
    archiveArtifacts artifacts: 'postfix_exporter', fingerprint: true
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
