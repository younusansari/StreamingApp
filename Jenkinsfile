// Define pipeline type
pipeline {
    agent any  // Use any available Jenkins agent
    
    
    // Step 1: Configure build triggers
    triggers {
        githubPush()  // Trigger on git push
        
    }
    
    // Step 2: Define pipeline stages
    stages {
        stage('Build') {
            steps {
            echo 'Building...'
            sh'''
            docker-compose build 
            '''
            }
        }
    }
}
