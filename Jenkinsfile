pipeline {
    agent any
    environment {
        SONAR_HOME = tool "Sonar"
    }
    
    stages {
        stage("Clone Code from GitHub") {
            steps {
                script {
                    echo "Code cloning from Github"
                    git(url: "https://github.com/qasimnauman/wanderlust.git", branch: "devops")
                }
            }
        }
        stage("SonarQube Quality Analysis") {
            steps {
                script {
                    echo "SonarQube quality analysis"
                    withSonarQubeEnv("Sonar") {
                        sh "$SONAR_HOME/bin/sonar-scanner -Dsonar.projectName=wanderlust -Dsonar.projectKey=wanderlust"
                    }
                }
            }
        }
        stage('OWASP Dependency Check') {
            steps {
                script {
                    echo 'Dependency Check using OWASP'
                    dependencyCheck additionalArguments: '--scan ./', odcInstallation: 'OWASP-Dependency Check'
                    dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
                }
            }
        }
        stage('Sonar Quality Gate Scan') {
            steps {
                timeout(time: 2, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: false
                }
            }
        }
        stage('Trivy Filesystem scan') {
            steps {
                script {
                    echo 'Scanning file system'
                    sh 'trivy fs --format table -o trivy-fs-report.html .'
                }
            }
        }
        stage('Deploy using Docker') {
            steps {
                script {
                    echo 'Deploying the app'
                    sh "docker-compose down && docker-compose up -d"
                }
            }
        }
    }
}
