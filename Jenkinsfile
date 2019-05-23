pipeline {
    agent {
		node {
        		label 'master'
        		customWorkspace '/var/lib/jenkins'
    		}
	}
    stages {
        stage('One') {
                steps {
                        //echo 'Demo pipeline job'
			//git url: 'https://github.com/shishaktkumar/calculator.git'
			checkout([$class: 'GitSCM', branches: [[name: '*/master']], doGenerateSubmoduleConfigurations: false, extensions: [[$class: 'RelativeTargetDirectory', relativeTargetDir: 'Demo']], submoduleCfg: [], userRemoteConfigs: [[url: 'https://github.com/shishaktkumar/calculator.git']]])
                }
        }
	    stage('Two'){
		    
		steps {
			input('Do you want to proceed?')
        }
	    }
        stage('Three') {
                when {
                        not {
                                branch "master"
                        }
                }
                steps {
			echo "Hello"
                        }
        }
        stage('Four') {
                parallel {
                        stage('Unit Test') {
                                steps{
                                        echo "Running the unit test..."
                                }
                        }
                        stage('Integration test') {
                        agent {
                                docker {
                                        reuseNode true
					image 'ubuntu'
                                        }
			}
				steps {
					echo 'Running the integration test..'
				}
                               
			}  }
        }
    }
}

