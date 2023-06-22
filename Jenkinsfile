def containerName = "medicure"
def tag = "latest"
def dockerHubUser = "sejalmm06"
def httpPort = "8082"
node {

    stage('Checkout') {
        checkout scm
    }

    stage('Build'){
        sh "mvn clean package"
    }
    
    stage('Publish Test Reports') {
        publishHTML([
            allowMissing: false,
            alwaysLinkToLastBuild: false,
            keepAll: false,
            reportDir: '/var/lib/jenkins/workspace/Finance-Me/target/surefire-reports',
            reportFiles: 'index.html',
            reportName: 'HTML Report',
            reportTitles: '',
            useWrapperFileDirectly: true
        ])
    }

    stage("Image Prune"){
         sh "docker image prune -a -f"
    }

    stage('Image Build'){
        sh "docker build -t $containerName:$tag --no-cache ."
        echo "Image build complete"
    }

    stage('Push to Docker Registry'){
        withCredentials([usernamePassword(credentialsId: 'dockerHubAccount', usernameVariable: 'dockerUser', passwordVariable: 'dockerPassword')]) {
            sh "docker login -u $dockerUser -p $dockerPassword"
            sh "docker tag $containerName:$tag $dockerUser/$containerName:$tag"
            sh "docker push $dockerUser/$containerName:$tag"
            echo "Image push complete"
        }
    }
        
stage('Run On Test Server') {
        ansiblePlaybook credentialsId: 'private-key', disableHostKeyChecking: true, installation: 'ansible', playbook: 'ansible-playbook.yml', inventory: 'hosts'
        
    }
    
node('172.31.13.114 (kubernetes-master)'){
  stage('Run on Prod Server') {
     sh "sudo kubectl apply -f deployment.yml"
     sh "sudo kubectl apply -f service.yml"
     sh "sudo kubectl get pods"
     sh "sudo kubectl get deployments"
     sh "sudo kubectl get services"
     sh "sudo kubectl get replica sets"
     echo "Deployment and service applied successfully"


    }
}
}
