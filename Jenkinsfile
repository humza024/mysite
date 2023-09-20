node {
  stage('SCM') {
    checkout scm
  }
  stage('SonarQube Analysis') {
    def scannerHome = tool 'sonarqube';
    withSonarQubeEnv() {
      sh "${scannerHome}/bin/sonar-scanner"
    }
  }
        stage('Pull Request') {
            steps {
                sh 'ssh -T -o StrictHostKeyChecking=no jenkins@162.55.3.153 "cd /var/www/html/development/Wintermads/Jenkins/backend && sudo git pull "'
                git branch: 'dev', credentialsId: 'eff36c60-5c78-4313-a542-b76ed68e74a2', url: 'https://github.com/mintmyne/wintermads-backend.git'
            }
        }
      //  stage('PHPUnit Wintermads') {
        //    steps {
          //      sh 'ssh -T -o StrictHostKeyChecking=no jenkins@162.55.3.153 "cd /var/www/html/development/Wintermads/Jenkins/backend && sudo php ./vendor/bin/phpunit"'
        //    }
    //    }
        //  stage('PHPStan Wintermds') {
        // steps {
        //     sh 'ssh -T -o StrictHostKeyChecking=no jenkins@162.55.3.153 "cd /var/www/html/development/Wintermads/Jenkins/backend &&  php vendor/bin/phpstan analyse app -l 0 --memory-limit 1G"'
        // }
        //    }
        stage('Deploy') {
            steps {
                sh 'ssh -T -o StrictHostKeyChecking=no jenkins@162.55.3.153 "cd /var/www/html/development/Wintermads/backend && sudo git pull && sudo php artisan optimize:clear"'
                git branch: 'dev', credentialsId: 'eff36c60-5c78-4313-a542-b76ed68e74a2', url: 'https://github.com/mintmyne/wintermads-backend.git'
            }
        }
        stage('Alert') {
            steps {
              script {
                def environmentName = " Wintermads Dev Backend"
                def webhookUrl = "https://hooks.slack.com/services/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"
                def gitCommit = sh(returnStdout: true, script: "git log -1 --pretty=format:'%h - %s (%an, %ae)'")
                def gitBranch = sh(returnStdout: true, script: "git rev-parse --abbrev-ref HEAD")
                def gitAuthor = sh(returnStdout: true, script: "git log -1 --pretty=format:'%an'")
                def gitEmail = sh(returnStdout: true, script: "git log -1 --pretty=format:'%ae'")
                def gitMessage = sh(returnStdout: true, script: "git log -1 --pretty=format:'%s'")
          
                def buildStatusText = "Build successful ✅"
                def color = "#36a64f"
                def changes = sh(returnStdout: true, script: "git diff --name-only HEAD HEAD~1")
                def changesText = changes.readLines().join(', ')
          
                sh """
                  curl -X POST \
                    -H 'Content-type: application/json; charset=utf-8' \
                    --data '{
                      \"channel\": \"#mychannel\",
                      \"username\": \"superbot\",
                      \"icon_emoji\": \":bot:\",
                      \"attachments\": [
                        {
                          \"color\": \"${color}\",
                          \"text\": \"${buildStatusText}\",
                          \"fields\": [
                            {
                              \"title\": \"Environment\",
                              \"value\": \"${environmentName}\",
                              \"short\": false
                            },
                            {
                              \"title\": \"Commit\",
                              \"value\": \"${gitCommit}\",
                              \"short\": false
                            },
                            {
                              \"title\": \"Branch\",
                              \"value\": \"${gitBranch}\",
                              \"short\": true
                            },
                            {
                              \"title\": \"Author\",
                              \"value\": \"${gitAuthor} <${gitEmail}>\",
                              \"short\": true
                            },
                            {
                              \"title\": \"Message\",
                              \"value\": \"${gitMessage}\",
                              \"short\": false
                            },
                            {
                              \"title\": \"Changes\",
                              \"value\": \"${changesText}\",
                              \"short\": false
                            }
                          ]
                        }
                      ]
                    }' \
                    ${webhookUrl}
                """
              }
            }
          }
}
