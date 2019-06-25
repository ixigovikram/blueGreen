//Jenkinsfile
node {
  stage('Preparation') {	
    //Installing kubectl in Jenkins agent
    sh 'curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl'
	sh 'chmod +x ./kubectl && mv kubectl /usr/local/bin'

	//Clone git repository
	git url:'https://github.com/sunnynew/blueGreen.git'
  }

  stage('Get Current Deployment') {
     withKubeConfig([credentialsId: 'jenkins-deployer-credentials', serverUrl: 'https://10.0.5.191:6443']) {
        //sh 'Check existing deployment -- BLUE, GREEN or first time deployment'
        sh(returnStdout: true, script: '''#!/bin/bash
            if [ `kubectl get deploy -n development|grep -c nodejs-deployment` -lt 1 ];then
                  DEPLOY=GREEN
            else
               CURR_DEPLOY=`kubectl get deploy -n development|grep nodejs-deployment|awk '{print $1}'|awk -F'-' '{print $NF}'`
                if [ $CURR_DEPLOY == "GREEN" ]; then
                        DEPLOY=BLUE
                  else
                        DEPLOY=GREEN
                fi
            fi
        '''.stripIndent())
        //Modify deployment.yaml for latest version.
        sh "echo $DEPLOY"
        sh "sed -i 's|COLOR|${DEPLOY}|g' deploy/deployment.yaml"
  }
}
  stage('Deploy latest version') {
     withKubeConfig([credentialsId: 'jenkins-deployer-credentials', serverUrl: 'https://10.0.5.191:6443']) { 
    	sh 'kubectl create cm nodejs-app --from-file=src/ --namespace=development -o=yaml --dry-run > deploy/cm.yaml'
     	sh 'kubectl apply -f deploy/deployment.yaml --namespace=development'
     	sh 'kubectl apply -f deploy/cm.yaml --namespace=development'

        //sh 'Check if SVC already deployed'
    	sh(returnStdout: true, script: '''#!/bin/bash
            if [ `kubectl get svc -n development|grep -c nodejs-service` -lt 1 ];then
               sh 'kubectl apply -f deploy/service.yaml --namespace=development'
            fi
        '''.stripIndent())
    }
  }
  stage('Patch Service to latest version') {
     withKubeConfig([credentialsId: 'jenkins-deployer-credentials', serverUrl: 'https://10.0.5.191:6443']) {
     echo "Creating k8s resources..."
     sleep 180
     sh "echo $DEPLOY"
     DESIRED= sh (
              script: "kubectl get deploy -n development | grep nodejs-deployment-${DEPLOY} | awk '{print \$2}' | grep -v DESIRED",
              returnStdout: true
         ).trim()
     CURRENT= sh (
              script: "kubectl get deploy -n development | grep nodejs-deployment-${DEPLOY} | awk '{print \$3}' | grep -v CURRENT",
              returnStdout: true
         ).trim()

	sh "Desired pods -- $DESIRED"
	sh "Current pods -- $CURRENT"
        //sh 'Patch service if DESIRED pods eq CURRENT pods'
        sh(returnStdout: true, script: '''#!/bin/bash
            if [ $DESIRED -eq $CURRENT ];then
                 sh """kubectl patch svc nodejs-service -n development -p '{\"spec\":{\"selector\":{\"app\":\"nodejs\",\"lable\":\"${DEPLOY}\"}}}'"""
            fi
        '''.stripIndent())

    }
  }
}
