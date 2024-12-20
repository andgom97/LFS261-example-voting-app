pipeline{

    agent none

    stages {
        stage("build-worker") {
            when{
                changeset '**/worker/**'
            }

            agent{
                docker {
                    image 'maven:3.9.8-sapmachine-21'
                    args '-v $HOME/.m2:/root/.m2'
                }
            }
            steps{
                echo 'Compiling worker app...'
                sh 'docker --version'
                dir('worker') {
                    sh 'mvn compile'
                }
            }
        }
        stage('build-result') {
            when{
                changeset "**/result/**"
            }
            steps{
                echo 'Compiling Node app'
                dir('result') {
                    sh 'npm install'
                }
            }
        }
        stage('build-test'){ 
            when{
                changeset "**/vote/**"
            }
            agent{
                docker{
                    image 'python:3.11-slim'
                    args '--user root'
                }
            }
            steps{ 
                echo 'Compiling vote app.' 
                dir('vote'){
            
                        sh "pip install -r requirements.txt"

                } 
            } 
        } 
        stage("test-worker") {
            when{
                changeset '**/worker/**'
            }
            agent{
                docker {
                    image 'maven:3.9.8-sapmachine-21'
                    args '-v $HOME/.m2:/root/.m2'
                }
            }
            steps{
                echo 'Running Unit Tests on worker app...'
                dir('worker') {
                    sh 'mvn clean test'
                }
            }
        }
        stage('test-result') {
            when{
                changeset "**/result/**"
            }
            steps{
                echo 'Running Unit Tests on Node app'
                dir('result') {
                    sh 'npm install'
                    sh 'npm test'
                }
            }
        }
        stage('test-vote'){ 
            when{
                changeset "**/vote/**"
            }
            agent {
                docker{
                    image 'python:3.11-slim'
                    args '--user root'
                }
            }
            steps{ 
                echo 'Running Unit Tests on vote app.' 
                dir('vote'){ 
                   
                        sh "pip install -r requirements.txt"
                        sh 'nosetests -v'
                        
                        
                } 
            } 
        } 
        stage("package-worker") {
            when{
                branch 'master'
                changeset '**/worker/**'
            }
            agent{
                docker {
                    image 'maven:3.9.8-sapmachine-21'
                    args '-v $HOME/.m2:/root/.m2'
                }
            }
            steps{
                echo 'Packaging worker app...'
                dir('worker') {
                    sh 'mvn package -DskipTests'
                    archiveArtifacts artifacts: '**/target/*.jar', fingerprint: true
                }
            }
        }
        stage('docker-package-worker'){
            agent any
            when{
                branch 'master'
                changeset '**/worker/**'
            }
            steps{
                echo'Packaging worker app with docker'
                script{
                    docker.withRegistry('https://index.docker.io/v1/','dockerlogin'){
                        def workerImage = docker.build("andgomalf/worker:v${env.BUILD_ID}","./worker")
                        workerImage.push()
                        workerImage.push("${env.BRANCH_NAME}")
                        workerImage.push("latest")
                    }
                }
            }
        }
        stage('docker-package-vote'){
            agent any
         
            steps{
                echo 'Packaging wvoteorker app with docker'
                script{
                    docker.withRegistry('https://index.docker.io/v1/', 'dockerlogin') {
                        // ./vote is the path to the Dockerfile that Jenkins will find from the Github repo
                        def voteImage = docker.build("okapetanios/vote:v${env.BUILD_ID}", "./vote")
                        voteImage.push()
                        voteImage.push("${env.BRANCH_NAME}")
                        voteImage.push("latest")
                    }
                }
            }
        }
      
    }

    post {
        always{
            echo 'Building multibranch pipeline for worker is completed..'
        }
    }
}
