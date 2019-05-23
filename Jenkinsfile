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
	    stage('Copying_Artifacts'){
		    steps {
			    sh '''
			    	## Copying artifacts to the destination
				cd $WORKSPACE/Control_Panel
				cp -rf message-viewer/target/message-viewer-0.3.war package/binaries/services/message-viewer.war
				cp -rf event-inspector/target/event-inspector-0.1.war package/binaries/services/event-inspector.war
				cp -fr ui/dist/ package/binaries/ui/
				#cp -rf /tmp/gen.conf .
				#cd $WORKSPACE/Control_Panel/package/binaries/scripts/ ; bash stop.sh; bash start.sh
				#cd $WORKSPACE
				#chmod 777 Control_Panel
				#cd $WORKSPACE/Control_Panel
				#docker run --network=host -v $(pwd):/zap/wrk/:rw -t owasp/zap2docker-weekly zap-baseline.py -t http://$(ip -f inet -o addr show docker0 | awk '{print $4}' | cut -d '/' -f 1):8080 -c gen.conf -r vulnerability_report.html
				#cp vulnerability_report.html reports/ 
				cd $WORKSPACE/Control_Panel
				mv reports package/
				mkdir artifacts
				cd $WORKSPACE/Control_Panel/package
				tar -czf control-panel.tar.gz * --exclude "finalzip.sh" --exclude "reports/email-content.txt" --exclude "reports/server-report.sh"
				mv control-panel.tar.gz ../artifacts/
				'''
		    }
		}
	    stage('Control_Panel_Fetch') {
                steps {
          		checkout([$class: 'GitSCM', branches: [[name: '*/master']], doGenerateSubmoduleConfigurations: false, extensions: [[$class: 'RelativeTargetDirectory', relativeTargetDir: 'bom-cylc']], submoduleCfg: [], userRemoteConfigs: [[url: 'git@github.com:ivorblockley/bom-cylc.git']]])
			sh 'pwd'
                }
        }
	    stage('Building_Cylc_PBS'){
		    steps {
			    sh '''
			    	ls 
				pwd
				rm -rf {cylc,example_pb_strings,kafka-testers,sms}
				mkdir pbs_ingestors
				mv $WORKSPACE/pbs_cylc_* pbs_ingestors/
				mv $WORKSPACE/pbs/* pbs_ingestors/
				mkdir cylc_ingestors
				mv $WORKSPACE/cylc_plugin/* cylc_ingestors/
				cd $WORKSPACE
				rm -rf {pbs,cylc_plugin}
			
				##Genrating pylint report for PBS_Prodcuer
				cd $WORKSPACE/pbs_ingestors/producer/bin
				touch __init__.py
				export PYTHONPATH=$WORKSPACE/pbs_ingestors/producer:$WORKSPACE/pbs_ingestors/consumer
				#pylint $(find . -maxdepth 5 -name "*.py" -print) --msg-template="{path}:{line}: [{msg_id}({symbol}), {obj}] {msg}" > ../pylint_pbs_producer.log || true
				##Generating Pylint report for PBS_Producer
				mkdir ../reports
				pylint producer --ignore=tests --msg-template="{path}:{line}: [{msg_id}({symbol}), {obj}] {msg}" > ../reports/pylint.log || true
				rm -f __init__.py
				cd ../reports
				PBS_PROD_PYLINT_STATUS=`tail -n 2 pylint.log  | awk -F "at " '{print $2}'  | cut -d '/' -f1`
				echo "PBS_PROD_PYLINT_STATUS=$PBS_PROD_PYLINT_STATUS" > $WORKSPACE/customvarilables.properties
				##Genrating Bandit report for PBS_Producer
				cd $WORKSPACE/pbs_ingestors/producer
				bandit -r $(find . -maxdepth 5 -name "*.py" -print) -x tests/ > reports/bandit.log || true
				echo "### Bandit summarized reports for PBS_Producer. For full report, please go through the log file. ###\n"
				grep -i "Run Metrics:" -A 20 reports/bandit.log
				#Severity_Undefined=`grep -i 'Run metrics:'  -A 50 bandit_cylc_ingestors.log  | grep -i 'Total issues (by severity):' -A 10 | grep -i 'Total issues (by confidence):' -B 50 | awk -F "Undefined:" '{print $2}'`
				#Severity_Low=`grep -i 'Run metrics:'  -A 50 bandit_cylc_ingestors.log  | grep -i 'Total issues (by severity):' -A 10 | grep -i 'Total issues (by confidence):' -B 50 | awk -F "Low:" '{print $2}'`
				#Severity_Medium=`grep -i 'Run metrics:'  -A 50 bandit_cylc_ingestors.log  | grep -i 'Total issues (by severity):' -A 10 | grep -i 'Total issues (by confidence):' -B 50 | awk -F "Medium:" '{print $2}'`
				#Severity_High=`grep -i 'Run metrics:'  -A 50 bandit_cylc_ingestors.log  | grep -i 'Total issues (by severity):' -A 10 | grep -i 'Total issues (by confidence):' -B 50 | awk -F "High:" '{print $2}'`


				##Generating pylint report for PBS_Consumer
				cd $WORKSPACE/pbs_ingestors/consumer/bin
				mkdir ../reports
				touch __init__.py
				pylint consumer --ignore=tests --msg-template="{path}:{line}: [{msg_id}({symbol}), {obj}] {msg}" > ../reports/pylint.log || true
				rm -f __init__.py
				cd ../reports
				PBS_CONS_PYLINT_STATUS=`tail -n 2 pylint.log  | awk -F "at " '{print $2}'  | cut -d '/' -f1`
				echo "PBS_CONS_PYLINT_STATUS=$PBS_CONS_PYLINT_STATUS" >> $WORKSPACE/customvarilables.properties
				##Genrating Bandit report for PBS_Consumer
				cd $WORKSPACE/pbs_ingestors/consumer
				bandit -r $(find . -maxdepth 5 -name "*.py" -print) -x tests/ > reports/bandit.log || true
				echo "### Bandit summarized reports for PBS_Consumer. For full report, please go through the log file. ###"\n
				grep -i "Run Metrics:" -A 20 reports/bandit.log

				##Generating pylint reports for pbs_cylc_libs
				cd $WORKSPACE/pbs_ingestors/pbs_cylc_libs
				mkdir reports
				pylint pbs_cylc_libs  '--msg-template={path}:{line}: [{msg_id}({symbol}), {obj}] {msg}' > reports/pylint.log || true
				cd reports
				PBS_CYLC_LIBS_PYLINT_STATUS=`tail -n 2 pylint.log  | awk -F "at " '{print $2}'  | cut -d '/' -f1`
				echo "PBS_CYLC_LIBS_PYLINT_STATUS=$PBS_CYLC_LIBS_PYLINT_STATUS" >> $WORKSPACE/customvarilables.properties
				##Generating bandit reports for pbs_cylc_libs
				cd ..	
				bandit -r $(find . -maxdepth 5 -name "*.py" -print) > reports/bandit.log
				grep -i "Run Metrics:" -A 20 reports/bandit.log
				
				##Generating pylint reports for CYLC_Ingestors
				cd $WORKSPACE/cylc_ingestors/
				export PYTHONPATH=$PYTHONPATH:$WORKSPACE/cylc_ingestors
				mkdir reports
				#pylint $(find . -maxdepth 5 -name "*.py" -print) --msg-template="{path}:{line}: [{msg_id}({symbol}), {obj}] {msg}" > pylint_cylc_ingestors.log || true
				pylint  cylc_ingestors  --msg-template="{path}:{line}: [{msg_id}({symbol}), {obj}] {msg}" > reports/pylint.log || true
				cd reports
				CYLC_ING_PYLINT_STATUS=`tail -n 2 pylint.log  | awk -F "at " '{print $2}'  | cut -d '/' -f1`
				echo "CYLC_ING_PYLINT_STATUS=$CYLC_ING_PYLINT_STATUS" >> $WORKSPACE/customvarilables.properties
				##Genrating Bandit report for CYLC_Ingestors
				cd $WORKSPACE/cylc_ingestors
				bandit -r $(find . -maxdepth 5 -name "*.py" -print) -x tests/ > reports/bandit.log || true
				echo "### Bandit summarized reports for CYLC_Ingestors. For full report, please go through the log file. ###\n"
				grep -i "Run Metrics:" -A 20 reports/bandit.log

				cd $WORKSPACE
				tar -czf cylc_ingestors.tar.gz cylc_ingestors
				tar -czf pbs_ingestors.tar.gz pbs_ingestors
				tar -czf reporting-db.tar.gz reporting-db
				'''
		    }
	    }
	    stage('Parent_Build'){
		    steps{
			checkout([$class: 'GitSCM', branches: [[name: '*/master']], doGenerateSubmoduleConfigurations: false, extensions: [[$class: 'RelativeTargetDirectory', relativeTargetDir: 'bom-schema']], submoduleCfg: [], userRemoteConfigs: [[url: 'git@github.com:ivorblockley/bom-schema.git']]])
			checkout([$class: 'GitSCM', branches: [[name: '*/master']], doGenerateSubmoduleConfigurations: false, extensions: [[$class: 'RelativeTargetDirectory', relativeTargetDir: 'pbs_cylc_textfiles']], submoduleCfg: [], userRemoteConfigs: [[url: 'git@github.com:PBSWorks/Altair-Weather-Suite.git']]])
			sh 'pwd'
			
		    }
	    }
	    stage('Build_Packages_Artifacts'){
		    steps {
			    sh '''
			    	#!/bin/bash
				echo $PWD
				id
				ls -la
				wget https://github.com/cylc/cylc/releases/ --no-check-certificate -O cylc.html
				#CYLC_VER=`cat cylc.html  | grep cylc-"[0-9].[0-9].[0-9]" | head -n 1 | awk -F 'cylc-' '{print $2}' | cut -d '<' -f1`
				##CYLC_VER=`cat cylc.html  | grep cylc-"[0-9].[0-9].[0-9]" | head -n 1 | awk -F 'cylc-' '{print $2}' | cut -d '/' -f4 | cut -d '"' -f1`
				CYLC_VER=7.8.1
				wget https://github.com/cylc/cylc/archive/$CYLC_VER.tar.gz --no-check-certificate  -O cylc-$CYLC_VER.tar.gz
				#sed -i "s/\(CYLC_VER[^0-9]*\)[0-9]*.[0-9]*.[0-9]*/CYLC_VER=$CYLC_VER/" cylcpbs/install.sh
				#sed -i "s/CYLC_REL_VER/$CYLC_VER/g" cylcpbs/install.sh
				wget https://github.com/metomi/rose/releases/ --no-check-certificate -O rose.html
				##ROSE_VER=`cat rose.html  | grep "Rose [0-9][0-9]*.[0-9]*.[0-9]*" | head -n 1 | awk -F 'Rose ' '{print $2}' | cut -d '<' -f1`
				ROSE_VER=2019.01.0
				wget https://github.com/metomi/rose/archive/$ROSE_VER.tar.gz --no-check-certificate -O rose-$ROSE_VER.tar.gz
				#sed -i "s/\(ROSE_VER[^0-9]*\)[0-9]*.[0-9]*.[0-9]*/ROSE_VER=$ROSE_VER/" cylcpbs/install.sh
				#sed -i "s/ROSE_REL_VER/$ROSE_VER/g" cylcpbs/install.sh
				wget https://github.com/metomi/fcm/releases/ --no-check-certificate -O fcm.html
				#FCM_VER=`cat fcm.html  | grep "FCM [0-9][0-9]*.[0-9]*.[0-9]*" | head -n 1 | awk -F 'FCM ' '{print $2}' | cut -d '<' -f1`
				FCM_VER=2017.10.0
				wget https://github.com/metomi/fcm/archive/$FCM_VER.tar.gz --no-check-certificate -O fcm-$FCM_VER.tar.gz
				#wget https://52.187.233.165/build/ --no-check-certificate -O cp.html
				#CP_VER=`cat cp.html | grep "package-[0-9][0-9]*-[0-9]*-[0-9]*_[0-9]*" | tail -n 1 | awk -F'a href="/build/' '{print $2}' | awk -F 'package-' '{print $2}' | cut -d '/' -f1`	
				#wget https://52.187.233.165/build/package-$CP_VER/control-panel-linux-$CP_VER.zip --no-check-certificate
				#wget https://52.187.233.165/build/package-$CP_VER/reports-$CP_VER.zip --no-check-certificate
				## Compiling the bom_pb_schema
				cd $WORKSPACE
				cd bom_schema
				which protoc
				protoc --version
				./build.sh --python
				cd ..
				cp -r bom_schema/build/generated/source/proto/main/python/bom_pb_schema .
				#sed -i "s/\(FCM_VER[^0-9]*\)[0-9]*.[0-9]*.[0-9]*/FCM_VER=$FCM_VER/" cylcpbs/install.sh
				#sed -i "s/FCM_REL_VER/$FCM_VER/g" cylcpbs/install.sh
				#cp {cylc-$CYLC_VER.tar.gz,fcm-$FCM_VER.tar.gz,rose-$ROSE_VER.tar.gz} cylcpbs/
				#tar -czvf pbs_cylc.tar.gz cylcpbs
				tar -czf  bom_pb_schema.tar.gz bom_pb_schema
				timestamp=$(date +'%Y-%m-%d_%H-%M-%S')
				timestamp=`echo $timestamp | tr -d '-'|tr -d '_'`
				PACKAGE="Altair_Weather_Solution_V5"
				cd $WORKSPACE
				mkdir -p $timestamp/$PACKAGE
				mv {*.tar.gz,pbs_cylc_textfiles/README,pbs_cylc_textfiles/release_notes.txt} $timestamp/$PACKAGE
				mv pbs_cylc_textfiles/"Release($PACKAGE).docx" $timestamp
				cd $timestamp/
				tar -czf $PACKAGE.tar.gz $PACKAGE
				md5sum $PACKAGE.tar.gz > "$PACKAGE"_Checksum
				rm -rf $PACKAGE
				cd $WORKSPACE 
				rm -f cylc.html rose.html fcm.html cp.html
				rm -rf *tmp bom_schema bom_pb_schema pbs_ingestors cylc_ingestors pbs_cylc_textfiles 
				#mv !($timestamp) $timestamp
				echo "$timestamp"
			    '''
		    }
	    }
    }
