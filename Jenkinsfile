pipeline {
    agent any
    triggers {
        pollSCM "* * * * *"  // SCM polling
    }
    stages {
        stage('Build Application') {
            steps {
                echo '=== Building Petclinic Application ==='
                sh 'mvn -B clean package'  // No tests are run in this stage
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
                    // Capture the full commit hash and short commit hash
                    GIT_COMMIT_HASH = sh(script: "git log -n 1 --pretty=format:'%H'", returnStdout: true).trim()
                    SHORT_COMMIT = GIT_COMMIT_HASH.substring(0, 7)  // Get short commit hash
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
            // No test reports to archive since tests are disabled
            echo '=== No test results to archive ==='
        }
    }
}
