def label = "worker-${UUID.randomUUID().toString()}"

podTemplate(label: label, containers: [
  containerTemplate(name: 'maven', image: 'maven', command: 'cat', ttyEnabled: true),
  containerTemplate(name: 'docker', image: 'docker', command: 'cat', ttyEnabled: true),
  containerTemplate(name: 'kubectl', image: 'lachlanevenson/k8s-kubectl:v1.8.8', command: 'cat', ttyEnabled: true),
  containerTemplate(name: 'helm', image: 'lachlanevenson/k8s-helm:latest', command: 'cat', ttyEnabled: true)
],
volumes: [
  hostPathVolume(mountPath: '/var/run/docker.sock', hostPath: '/var/run/docker.sock')
]) {
  node(label) {
    def myRepo = checkout scm
    def gitCommit = myRepo.GIT_COMMIT
    def gitBranch = myRepo.GIT_BRANCH
    def shortGitCommit = "${gitCommit[0..10]}"
    def previousGitCommit = sh(script: "git rev-parse ${gitCommit}~", returnStdout: true)
    def PROJECT_NAME = "MY_JOB_NAME"
    
    
    stage('Checkout') {
      sh """
        echo ${PROJECT_NAME}
        git checkout master
        echo currentBuildNumber, ${currentBuild.number}
        echo gitCommit, ${gitCommit}
        echo gitBranch, ${gitBranch}
        echo shortGitCommit, ${shortGitCommit}
        echo previousGitCommit, ${previousGitCommit}
        """
    }
    stage('Build') {
      container('docker') {
        sh "docker build -t ${PROJECT_NAME}:${gitCommit} ."
      }
    }
    stage('Test') {
      container('docker') {
        sh "docker run --entrypoint ./test $PROJECT_NAME:${gitCommit}"
      }
    }
    stage('Push') {
      container('docker') {
        withCredentials([[$class: 'UsernamePasswordMultiBinding',
          credentialsId: 'dockerhub',
          usernameVariable: 'DOCKER_HUB_USER',
          passwordVariable: 'DOCKER_HUB_PASSWORD']]) {
          sh """
            docker login -u ${DOCKER_HUB_USER} -p ${DOCKER_HUB_PASSWORD}
            docker tag ${PROJECT_NAME}:${gitCommit} ${DOCKER_HUB_USER}/${PROJECT_NAME}:${gitCommit}
            docker push ${DOCKER_HUB_USER}/${PROJECT_NAME}:${gitCommit}
            docker rmi ${PROJECT_NAME}:${gitCommit} ${DOCKER_HUB_USER}/${PROJECT_NAME}:${gitCommit}
            """
        }
      }
    }
    stage('Run push') {
      withCredentials([[$class: 'UsernamePasswordMultiBinding',
          credentialsId: 'github',
          usernameVariable: 'GIT_USERNAME',
          passwordVariable: 'GIT_PASSWORD']]) {
          sh """
            ls -alh .
            echo "print('plz work')" >> test.py
            git config --global user.name ${GIT_USERNAME}
            git config --global user.email ${GIT_USERNAME}@gmail.com
            git commit -am 'msg'
            git log --graph --decorate
            git push https://${GIT_USERNAME}:${GIT_PASSWORD}@github.com/${GIT_USERNAME}/jenkins-pipeline-sample.git master
            """
        }
    }

  }
}
