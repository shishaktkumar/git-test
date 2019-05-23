pipeline {
    agent {
		node {
        		label 'master'
        		customWorkspace '/var/lib/jenkins/workspace/Pipeline_demo'
    		}
	}
    stages {
        stage('Control_Panel_Fetch') {
                steps {
          		checkout([$class: 'GitSCM', branches: [[name: '*/master']], doGenerateSubmoduleConfigurations: false, extensions: [[$class: 'RelativeTargetDirectory', relativeTargetDir: 'Control_Panel']], submoduleCfg: [], userRemoteConfigs: [[url: 'git@gitlab.pclm.altair.com:bom/pwp.git']]])
			sh 'pwd'
                }
        }
	stage('Building CP_1'){
		    
		steps {
			sh '''
				cd $WORKSPACE/Control_Panel
				mvn -v
				node -v
				npm -v
				ls -la
				## Building packages 
				echo "Building message viewer war." 
				cd message-viewer/
				mvn clean -Dmaven.test.skip=true package
				cd ..
				echo "Building event inspector war."
				cd event-inspector/
				mvn clean -Dmaven.test.skip=true package
				cd ..
				echo "Building UI code."
				cd ui/
				npm install
				npm run build
				'''
        		}
	    		}
        stage('Building CP_2') {
                steps {
			sh '''
			## Running code analysis on the package components
			cd $WORKSPACE/Control_Panel
			cd message-viewer/
			echo "Starting Code analysis on message viewer services."
			mvn pmd:pmd
			cd ..
			cd event-inspector/
			echo "Starting Code analysis on event inspector services."
			mvn pmd:pmd
			cd ..
			cd ui/
			echo "Starting static code analysis on UI."
			set +e
			npm run eslint-analysis || true
			'''
                        }
	        }
        stage('Testing CP_1') {
		steps {
			sh '''
			set -e
			## Running tests on the package coomponents
			cd $WORKSPACE/Control_Panel
			cd message-viewer/
			mvn test -Dspring.config.location=raju.properties | true
			cd ..
			cd reports/
			mkdir message-viewer-services-report/
			cd ..
			cp -f $(pwd)/message-viewer/reports/junit-test-report.html reports/message-viewer-services-report/raju-test-report.html
			cd message-viewer/
			mvn test -Dspring.config.location=david.properties | true 
			cd ..
			cp -f $(pwd)/message-viewer/reports/junit-test-report.html reports/message-viewer-services-report/david-test-report.html
			cp -f $(pwd)/message-viewer/target/site/pmd.html reports/message-viewer-services-report/services-code-analysis.html
			cd event-inspector/
			mvn test | true
			cd ..
			cd reports/
			mkdir event-inspector-services-report/
			cd ..
			cp -f $(pwd)/event-inspector/reports/junit-test-report.html reports/event-inspector-services-report/test-report.html
			cp -f $(pwd)/event-inspector/target/site/pmd.html reports/event-inspector-services-report/services-code-analysis.html
			cd ui/
			npm install
			npm run test
			cd ..
			cd reports/
			mkdir ui-report/
			cd ..
			cp -f $(pwd)/ui/reports/axe-report.html reports/ui-report/
			cp -f $(pwd)/ui/reports/ui-code-analysis.html reports/ui-report/ui-code-analysis.html
			'''
        }
    }
}

