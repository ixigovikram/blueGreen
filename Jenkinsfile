//Jenkinsfile
node {
  stage('Preparation') {
    //Installing kubectl in Jenkins agent
    sh 'curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl'
	sh 'chmod +x ./kubectl && mv kubectl /usr/local/bin'

	//Clone git repository
	git url:'https://bitbucket.org/advatys/jenkins-pipeline.git'
  }

  stage('Production') {
    withKubeConfig([credentialsId: 'jenkins-deployer-credentials', serverUrl: 'https://104.155.31.202']) {
      
    	sh 'kubectl create cm nodejs-app --from-file=src/ --namespace=myapp-production -o=yaml --dry-run > deploy/cm.yaml'
     	sh 'kubectl apply -f deploy/ --namespace=myapp-production'
      
    }
  }
}
