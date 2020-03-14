node {
    stage('Checkout') {
        def scm =   checkout scm
    }
    stage('Build'){
        container('python:3.7') {
            python test.py
        }
    }
    stage('Build'){
        container('docker') {
            docker build . -t plz
        }
    }
}
