import jenkins.model.*

//branch2 test
//test PR new_feature

def label = env.JOB_NAME.replaceAll('\\s','_')

node("(swarm && deployed=${label}) || (swarm && !deployed)" ) {

  try {
    slackSend (color: '#FFFF00', message: "STARTED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})")

    checkout scm

    for (slave in jenkins.model.Jenkins.instance.slaves) {
        def oldLabelName = slave.getLabelString()
        def nodeName = slave.getNodeName()

        if (nodeName == env.NODE_NAME && oldLabelName.contains('deployed ') && !oldLabelName.contains("deployed=${label}")) {
	    currentBuild.result = "FAILED"
	    slackSend (color: '#FF0000', message: "FAILED: Job '${env.JOB_NAME} : Provided node already binded")
	    error "Job '${env.JOB_NAME}' : Provided node already binded, please try again"
        }
 
        if (nodeName == env.NODE_NAME && !oldLabelName.contains("deployed=${label}")) {
            newLabelName = "swarm" + " " + "deployed" + " " + "deployed=${label}"
            slave.setLabelString(newLabelName)
	    sleep time: 1, unit: 'MINUTES'
        }

        if (nodeName != env.NODE_NAME && oldLabelName.contains("deployed=${label}") && oldLabelName.contains('swarm')) {
            slave.setLabelString('swarm')
        }
        
        oldLabelName = null
        nodeName = null
        
    }

    def node_ip  = sh( script: 'hostname -I | grep -o "[0-9]\\{1,3\\}\\.[0-9]\\{1,3\\}\\.[0-9]\\{1,3\\}\\.[0-9]\\{1,3\\}" | head -1', returnStdout: true).trim()
    def work_dir = sh( script: 'pwd', returnStdout: true).trim()
    slackSend (color: '#FFFF00', message: "BINDED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' on node ${env.NODE_NAME}(${node_ip}):(${work_dir})")
    node_ip  = null
    work_dir = null

    // build and test steps
    sh script: 'DOCKER_HOST=tcp://localhost:4243 docker-compose kill ; docker-compose rm -f; docker-compose up -d'
    slackSend (color: '#00FF00', message: "SUCCESSFUL: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})")

  } catch (e) {
    currentBuild.result = "FAILED"
    slackSend (color: '#FF0000', message: "FAILED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})")
    throw e
  }

}

