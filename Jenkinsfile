node {
    
    def mavenHome
    def mavenCMD
    def docker
    def dockerCMD
    def tagName
    
    stage('Prepare the environment based on the global tools'){
        echo 'Initialize the variables'
        mavenHome = tool name: 'myMaven' , type: 'maven'
        mavenCMD = "${mavenHome}/bin/mvn"
        docker = tool name: 'myDocker' , type: 'org.jenkinsci.plugins.docker.commons.tools.DockerTool'
        dockerCMD = "${docker}/bin/docker"
        tagName="3.0"
        
    }
    stage('checkout the git repository'){
        echo 'Cloning the repository .................................'
        git 'https://github.com/niladrimondal/insure-me.git'
        
    }
    stage('package the insume application'){
        echo 'clean ... Compile....Test....Package....'
        sh "${mavenCMD} clean package"
    }
    stage('Publish the Test report'){
        echo 'publish the test reports'
        publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: '/var/lib/jenkins/workspace/insureme-deployment-pipeline/target/surefire-reports', reportFiles: 'index.html', reportName: 'HTML Report', reportTitles: '', useWrapperFileDirectly: true])
        
    }
    stage('Build the Docker Image using the Docker file'){
        try{
        echo 'Building the Docker Image'    
        sh "${dockerCMD} build -t niladrimondaldcr/insureme:${tagName} ."
        }
        catch(Exception e){
            echo 'Exception Occur in Stage Docker Biuld'
            currentBuild.result = "FAILURE"
            emailext body: '''Hello 

            We are having issue with the Docker Build Command . Can you please look into the same.

            Thanks,
            Jenkins Admin''', subject: 'Attention: ${JOB_NAME} is failed. Please look into the ${BUILD_NUMBER} ', to: 'niladrimondal.mondal@gmail.com'
        }
    }
    stage('Push the Docker image'){
        echo 'push the Docker Image to the dockerhub'
        withCredentials([string(credentialsId: 'DockerPassword', variable: 'dockerpassword')]) {
                sh "${dockerCMD} login -u niladrimondaldcr -p ${dockerpassword}"
                sh "${dockerCMD} push niladrimondaldcr/insureme:${tagName}"
        }
        
    }
    stage('configure the application in the Test Server using Ansible'){
        echo 'deploy application on test server'
        ansiblePlaybook become: true, credentialsId: 'ansiblekey', disableHostKeyChecking: true, installation: 'myAnsible', inventory: '/etc/ansible/hosts', playbook: 'ansible-playbook.yml'
        
        
    }
}