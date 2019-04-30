pipeline {
    agent any
    stages {
        stage('One') {
                steps {
                        echo 'Hi, this is Zulaikha from edureka'
			git https://github.com/shishaktkumar/calculator.git
			
                }
        }
	stage('Checkout: Code') {
		steps {
			git url 'https://github.com/shishaktkumar/calculator.git'
			//sh "mkdir -p $WORKSPACE/repo;\
			//git config --global user.email 'email@address.com';\
			//git config --global user.name 'myname';\
			//git config --global push.default simple;\
			//git clone $BUILD_SCRIPTS_GIT repo/$BUILD_SCRIPTS"
			//sh "chmod -R +x $WORKSPACE/repo/$BUILD_SCRIPTS"
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
                                        reuseNode false
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

