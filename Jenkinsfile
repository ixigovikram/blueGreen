//Jenkinsfile
node {
  stage('Preparation') {	
    //Installing kubectl in Jenkins agent
    sh 'curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl'
	sh 'chmod +x ./kubectl && mv kubectl /usr/local/bin'

	//Clone git repository
	git url:'https://github.com/sunnynew/blueGreen.git'
  }

    stage('Creating development Namespace') {
    //Verify kubectl in Jenkins agent
    withKubeConfig([credentialsId: 'jenkins-deployer-credentials', serverUrl: 'https://10.0.5.191:6443']) {
      
    	//sh 'kubectl create ns development'
    	bash(returnStdout: true, script: '''#!/bin/bash
            if [ `kubectl get ns|grep development|wc -l` -lt 1 ];then
               kubectl create ns development
               DEPLOY="HELLO"
	       echo "${DEPLOY}"
	    else
	       echo "Already Created."
               DEPLOY="HI"
	       echo "${DEPLOY}"
            fi
        '''.stripIndent())
    }
  }

  stage('Get Current Deployment') {
     withKubeConfig([credentialsId: 'jenkins-deployer-credentials', serverUrl: 'https://10.0.5.191:6443']) {
        //sh 'Check existing deployment -- BLUE, GREEN or first time deployment'
        sh(returnStdout: true, script: '''#!/bin/bash
            if [ `kubectl get deploy -n development|grep -c nodejs-deployment` -lt 1 ];then
                  DEPLOY="GREEN"
            else
               CURR_DEPLOY=`kubectl get deploy -n development|grep nodejs-deployment|awk '{print $1}'|awk -F'-' '{print $NF}'`
                if [ $CURR_DEPLOY == "GREEN" ]; then
                        DEPLOY="BLUE"
                  else
                        DEPLOY="GREEN"
                fi
            fi
        '''.stripIndent())
        //Modify deployment.yaml for latest version.
        sh "echo $DEPLOY"
        sh "sed -i 's|COLOR|${DEPLOY}|g' deploy/deployment.yaml"
  }
}
}
