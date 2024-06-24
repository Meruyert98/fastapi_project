pipeline {
    
    agent any;
    
    environment {
        // Telegram configuration
        TOKEN = credentials('telegram-credentials')
        CHAT_ID = credentials('Telegram_ChatID')
        
        BUILD_NUMBER_ENV = "${env.BUILD_NUMBER}"
        TEXT_BUILD = "[#${env.BUILD_NUMBER}] Project: ${JOB_NAME}"
    }
    
    stages{
        
        stage('Git'){
            steps{
                git url:'https://github.com/Meruyert98/fastapi_project.git', branch:'main', credentialsId: 'meruyert'
            }
        }
        
        stage('Docker'){
            steps{
                sh 'docker build -t pythonproject .'
                
            }
        }
        
        stage('Build'){
            steps{
                echo 'Build'
                sh 'python3 main.py'
            }
        }
        
        stage('Test'){
            steps{
                echo 'Test'
                sh 'pytest test_main.py'
            }
        }
        
        stage('Run') {
            steps {
                script {
                    sh 'docker run -d -p 8083:8000 pythonproject'
                }
            }
        }
        
        stage('Deploy'){
            input{
                message "Do you want to proceed for production deployment?"
            }
            steps{
                echo "Deploy"
            }
        }
        
    }
    post {
	always {
            emailext (
                    body: '''<html>
					<body>
					    <p>Pipeline: ${PROJECT_NAME}</p>
						<p>Build Status: ${BUILD_STATUS}</p>
						<p>Build Number: ${BUILD_NUMBER}</p>
						<p>Check the <a href="${BUILD_URL}">console output</a> to view the results.</p>
						
					</body>
				</html>''', 
				    recipientProviders: [[$class: 'DevelopersRecipientProvider'], [$class: 'RequesterRecipientProvider']], 
				    subject: "Pipeline Status: ${BUILD_NUMBER}", 
				    mimeType: 'text/html'
				    )
        }
	success {
            script {
                sh "curl -X POST -H \"Content-Type: application/json\" -d \"{\\\"chat_id\\\":$CHAT_ID, \\\"text\\\": \\\"[SUCCESS] ✅Pipeline succeeded! \n$TEXT_BUILD\\\", \\\"disable_notification\\\": false}\" https://api.telegram.org/bot$TOKEN/sendMessage"
            }
    }
    failure {
            script {
                sh "curl -X POST -H \"Content-Type: application/json\" -d \"{\\\"chat_id\\\":$CHAT_ID, \\\"text\\\": \\\"[FAILED] ❌Pipeline failed! \n$TEXT_BUILD\\\", \\\"disable_notification\\\": false}\" https://api.telegram.org/bot$TOKEN/sendMessage"
            }
    }
    aborted {
            script {
                sh "curl -X POST -H 'Content-Type: application/json' -d '{\"chat_id\": \"${CHAT_ID}\", \"text\": \"${JOB_NAME}: #${BUILD_NUMBER}\n❌Deploy aborted! \n$TEXT_BUILD\", \"disable_notification\": false}' \"https://api.telegram.org/bot${TOKEN}/sendMessage\""
            }
        }  
}

    
}