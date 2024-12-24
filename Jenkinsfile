pipeline {
    agent any
    triggers {
        pollSCM "* * * * *"
    }
    stages {
        stage('Build Application') {
            steps {
                echo '=== Building Petclinic Application ==='
                // Disable tests by ensuring Surefire plugin does not execute any tests
                sh 'mvn -B clean package -DskipTests -Dtest='  // Ensure no tests are triggered
            }
        }
        stage('Build Docker Image') {
            when {
                branch 'master'
            }
            steps {
                echo '=== Building Petclinic Docker Image ==='
                script {
                    app = docker.build("jonathannkrumah/amazon-eks-jenkins-terraform")
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
                    GIT_COMMIT_HASH = sh(script: "git log -n 1 --pretty=format:'%H'", returnStdout: true).trim()
                    SHORT_COMMIT = GIT_COMMIT_HASH.substring(0, 7)
                    docker.withRegistry('https://registry.hub.docker.com', 'dockerHubCredentials') {
                        app.push("$SHORT_COMMIT")
                        app.push("latest")
                    }
                }
            }
        }
        stage('Remove local images') {
            steps {
                echo '=== Delete the local docker images ==='
                sh "docker rmi -f jonathannkrumah/amazon-eks-jenkins-terraform:latest || :"
                sh "docker rmi -f jonathannkrumah/amazon-eks-jenkins-terraform:$SHORT_COMMIT || :"
            }
        }
    }
    post {
        always {
            echo '=== No test results to archive ==='
        }
    }
}
