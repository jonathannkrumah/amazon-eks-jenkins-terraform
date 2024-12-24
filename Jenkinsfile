pipeline {
    agent any
    
    triggers {
        pollSCM('H/5 * * * *') // Adjust polling frequency to every 5 minutes
    }
    
    environment {
        MAVEN_OPTS = '-Xmx2048m -XX:MaxMetaspaceSize=512m'
    }
    
    stages {
        stage('Build Application') {
            steps {
                echo '=== Building Petclinic Application ==='
                sh 'mvn -B -DskipTests clean package'
            }
        }
        
        stage('Test Application') {
            steps {
                echo '=== Testing Petclinic Application ==='
                sh '''
                    export MAVEN_OPTS="$MAVEN_OPTS"
                    mvn test -Dsurefire.useSystemClassLoader=false
                '''
            }
            post {
                always {
                    echo '=== Publishing Test Results ==='
                    junit '**/target/surefire-reports/*.xml'
                }
            }
        }
        
        stage('Build Docker Image') {
            when {
                branch 'master'
            }
            steps {
                echo '=== Building Petclinic Docker Image ==='
                script {
                    app = docker.build("ibuchh/petclinic-spinnaker-jenkins")
                }
            }
        }
        
        stage('Push Docker Image') {
            when {
                branch 'master'
            }
            steps {
                echo '=== Pushing Petclinic Docker Image ==='
                script {
                    def gitCommitHash = sh(script: "git rev-parse HEAD", returnStdout: true).trim()
                    def shortCommit = gitCommitHash.take(7)
                    docker.withRegistry('https://registry.hub.docker.com', 'dockerHubCredentials') {
                        app.push("${shortCommit}")
                        app.push("latest")
                    }
                }
            }
        }
        
        stage('Remove Local Images') {
            when {
                branch 'master'
            }
            steps {
                echo '=== Deleting Local Docker Images ==='
                script {
                    try {
                        sh "docker rmi -f ibuchh/petclinic-spinnaker-jenkins:latest || true"
                        sh "docker rmi -f ibuchh/petclinic-spinnaker-jenkins:${shortCommit} || true"
                    } catch (Exception e) {
                        echo 'Image removal failed, but continuing pipeline execution'
                    }
                }
            }
        }
    }
    
    post {
        always {
            echo '=== Cleaning Workspace ==='
            cleanWs()
        }
    }
}
