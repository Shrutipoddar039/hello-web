pipeline {
    agent any
    tools {
        maven 'maven3'
        jdk 'jdk21'
    }
    stages {
        stage('Checkout') {
            steps { checkout scm }
        }
        stage('Build') {
            steps { bat 'mvn -B clean package -DskipTests' }
        }
        stage('Test') {
            steps { bat 'mvn -B test' }
            post {
                always { junit 'target/surefire-reports/*.xml' }
            }
        }
        stage('Deploy') {
            steps {
                deploy adapters: [tomcat9(credentialsId: 'tomcat-deployer-3', path: '', url: 'http://localhost:8081')],
                       contextPath: 'hello-web',
                       war: 'target/*.war'
            }
        }
    }
    post {
        success { echo 'Pipeline succeeded' }
        failure { echo 'Pipeline failed' }
        always  { echo 'Pipeline finished' }
    }
}