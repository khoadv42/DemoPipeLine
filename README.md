# Demo
pipeline {
    agent any 
    tools {
        // Must match the names set in Global Tool Configuration
        maven 'M3' 
        jdk 'JDK17'
    }
    stages {
        stage('Checkout Secondary Repo') {
            steps {
                // Định nghĩa SCM (Git) trực tiếp bằng cú pháp Groovy
                git branch: 'main', 
                    credentialsId: 'DemoPipeLine', 
                    url: 'https://github.com/khoadv42/DemoPipeLine.git'
            }
        }
        stage('Build and Test') {
            steps {
                // Run Maven goals using the configured Maven tool
                sh 'mvn clean install' 
            }
        }
    }
}