import jenkins.model.*

//branch4

def label = env.JOB_NAME.replaceAll('\\s','_')

node("(swarm && deployed=${label}) || (swarm && !deployed)" ) {
  try {
    slackSend (color: '#FFFF00', message: "STARTED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})")

    checkout scm

    for (slave in jenkins.model.Jenkins.instance.slaves) {
        oldLabelName = slave.getLabelString()
        nodeName = slave.getNodeName()
 
        if (nodeName == env.NODE_NAME && !oldLabelName.contains("deployed=${label}")) {
            newLabelName = "swarm" + " " + "deployed" + " " + "deployed=${label}"
            slave.setLabelString(newLabelName)
        }

        if (nodeName != env.NODE_NAME && oldLabelName.contains("deployed=${label}") && oldLabelName.contains('swarm')) {
            slave.setLabelString('swarm')
        }
        
    }

    node_ip  = sh( script: 'hostname -I | grep -o "[0-9]\\{1,3\\}\\.[0-9]\\{1,3\\}\\.[0-9]\\{1,3\\}\\.[0-9]\\{1,3\\}" | head -1', returnStdout: true).trim()
    work_dir = sh( script: 'pwd', returnStdout: true).trim()
    sh 'rm -f /home/jenkins/deploy ; ln -s /home/jenkins/deploy ${work_dir}'
    sh 'DOCKER_HOST=tcp://localhost:4243 docker-compose kill ; docker-compose rm ; docker-compose up -d'
    slackSend (color: '#FFFF00', message: "BINDED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' on node ${env.NODE_NAME}(${node_ip}):(/home/jenkins/deploy)")

    // build and test steps
    slackSend (color: '#00FF00', message: "SUCCESSFUL: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})")

  } catch (e) {
    currentBuild.result = "FAILED"
    slackSend (color: '#FF0000', message: "FAILED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})")
    throw e
  }

}
