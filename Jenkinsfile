pipeline {
    agent any
    stages {
        stage("Fetch") {
            steps {
               git "https://github.com/PranavMasekar/HashNode-Blogs.git"
            }
        }
        stage("Print File"){
            steps {
                sh 'echo pwd'
                sh 'cat EKS.md'
            }
        }
    }
}