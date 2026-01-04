pipeline {
    agent any

    stages {
        stage('Test') {
            steps {
                // Run tests
                bat './gradlew test'
            }
            post {
                always {
                    // Archive JUnit results
                    junit 'build/test-results/test/*.xml'
                    
                    // Generate Cucumber Reports (Requires "Cucumber reports" plugin installed)
                    cucumber buildStatus: 'UNSTABLE', 
                             jsonReportDirectory: 'build/reports/cucumber', 
                             fileIncludePattern: '**/*.json'
                }
            }
        }

        stage('Code Analysis') {
            steps {
                // 'sonar' must match the name in Manage Jenkins -> System -> SonarQube servers
                withSonarQubeEnv('sonar') { 
                    bat './gradlew sonarqube'
                }
            }
        }

        stage('Code Quality') {
            steps {
                timeout(time: 1, unit: 'HOURS') {
                    // Waits for SonarQube webhook response
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Build') {
            steps {
                bat './gradlew assemble javadoc'
                // Archive the Jar and Javadoc
                archiveArtifacts artifacts: 'build/libs/*.jar, build/docs/javadoc/**', fingerprint: true
            }
        }

        stage('Deploy') {
            steps {
                // Publishes to MyMavenRepo using Env Vars
                bat './gradlew publish'
            }
        }
    }

    post {
        success {
            // Note: Slack requires the "Slack Notification Plugin" configured
            slackSend color: 'good', message: "Build Success: ${env.JOB_NAME} [${env.BUILD_NUMBER}]"
            
            // Email Notification
            emailext body: "The build was successful. Artifacts deployed to MyMavenRepo.\n\nProject: ${env.JOB_NAME}\nBuild Number: ${env.BUILD_NUMBER}\nURL: ${env.BUILD_URL}", 
                     subject: "Build Success: ${env.JOB_NAME}", 
                     to: 'mm_maraf@esi.dz'
        }
        failure {
            slackSend color: 'danger', message: "Build Failed: ${env.JOB_NAME} [${env.BUILD_NUMBER}]"
            
            emailext body: "The build failed. Please check the logs.\n\nProject: ${env.JOB_NAME}\nBuild Number: ${env.BUILD_NUMBER}\nURL: ${env.BUILD_URL}", 
                     subject: "Build Failed: ${env.JOB_NAME}", 
                     to: 'mm_maraf@esi.dz'
        }
    }
}