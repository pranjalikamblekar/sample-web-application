def getDockerTag(){
    def tag = sh script: 'git rev-parse HEAD', returnStdout: true
    return tag
}

pipeline{
    agent any
    environment{
	    Docker_tag = getDockerTag()
    }
    stages{
        stage('Quality Gate Status Check'){
            agent {
                docker {
                    image 'maven'
                    args '-v $HOME/.m2:/root/.m2'
                }
            }
        
            steps{
                script{
                    withSonarQubeEnv('sonarserver') { 
                        sh "mvn sonar:sonar"
                    }
                    timeout(time: 1, unit: 'HOURS') {
                        def qg = waitForQualityGate()
                        if (qg.status != 'OK') {
                            error "Pipeline aborted due to quality gate failure: ${qg.status}"
                        }
                    }
		            sh "mvn clean install"
                }
            }
        }

        stage('build'){
            steps{
                script{
		            sh 'cp -r ../devops-training/target .'
                    sh 'docker build . -t pranjalikamblekar/devops-training:$Docker_tag'
		            withCredentials([string(credentialsId: 'Docker', variable: 'docker-password')]) {  
				        sh 'docker login -u pranjalikamblekar -p $docker-password'
				        sh 'docker push pranjalikamblekar/devops-training:$Docker_tag'
			        }
                }
            }
        }  
    }
}