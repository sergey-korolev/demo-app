import jenkins.model.*

def label = env.JOB_NAME.replaceAll('\\s','_')

node('swarm && deployed=${label} || swarm && !deployed*' ) {
  try {
    slackSend (color: '#FFFF00', message: "STARTED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})")

    checkout scm

    for (slave in jenkins.model.Jenkins.instance.slaves) {
        oldLabelName = slave.getLabelString()
        nodeName = slave.getNodeName()
 
        if (nodeName == env.NODE_NAME && !oldLabelName.contains("deployed=${label}")) {
            newLabelName = "swarm" + " " + "deployed=${label}"
            slave.setLabelString(newLabelName)
        }

        if (nodeName != env.NODE_NAME && oldLabelName.contains("deployed=${label}")) {
            slave.setLabelString('swarm')
            // cleanup step
            sh 'DOCKER_HOST=tcp://localhost:4243 docker-compose kill ; docker-compose rm'
            slackSend (color: '#FFFF00', message: "UNBINDED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' release node ${env.NODE_NAME})")
        }
        
    }

    slackSend (color: '#FFFF00', message: "BINDED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' on node ${env.NODE_NAME})")

    // build and test steps
    sh 'DOCKER_HOST=tcp://localhost:4243 docker-compose kill ; docker-compose rm ; docker-compose up -d'

    slackSend (color: '#00FF00', message: "SUCCESSFUL: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})")

  } catch (e) {
    currentBuild.result = "FAILED"
    slackSend (color: '#FF0000', message: "FAILED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})")
    throw e
  }

}
