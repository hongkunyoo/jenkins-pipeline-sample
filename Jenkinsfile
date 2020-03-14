node {
    stage('Checkout') {
        def scm =   checkout scm
    }
    stage('Test'){
        container('python:3.7') {
            sh "python test.py"
        }
    }
    stage('Build'){
        container('docker') {
            sh "docker build . -t plz"
        }
    }
}
