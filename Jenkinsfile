node {

    def controllerServer="54.68.70.26"
    def qaServer="54.188.60.175"
    def mvnHome=tool name: 'apache-maven-3.6.3', type: 'maven'

    stage('Git Checkout') { // for display purposes
        // Get some code from a GitHub repository
        git changelog: false, poll: false, url: 'https://github.com/akhil7000/DevopsBasics.git'
    }

    stage('Build') {
        // Run the maven build
        withEnv(["MVN_HOME=$mvnHome"]) {
            sh '"$MVN_HOME/bin/mvn" test install'
        }
    }

    stage('Create hosts file'){

        try{
            timeout(time: 60, unit: 'SECONDS') {
                qaServer = input message: 'Enter IP of QA server',
                        parameters: [string(defaultValue: "$qaServer", description: '', name: 'qaServer', trim: true)]
            }
        }
        catch (Exception e){
            echo 'Exception thrown due to timeout'
        }

        sh 'rm -rf hosts'
        sh 'touch hosts'
        sh 'echo \"localhost\" >> hosts'
        sh 'echo \"[QA]\" >> hosts'
        sh "echo \"ec2-user@${qaServer}\" >> hosts"
        sh 'cat hosts'
    }

    stage('Deploy on - Controller & QA Server'){
        sshPublisher(publishers:
                [sshPublisherDesc(configName: 'Application-Server', transfers:
                        [sshTransfer(cleanRemote: false, excludes: '', execCommand: 'ansible-playbook -i hosts remoteplaybook.yml --ssh-common-args=\'-o StrictHostKeyChecking=no\'',
                                execTimeout: 120000, flatten: false, makeEmptyDirs: false, noDefaultExcludes: false, patternSeparator: '[, ]+',
                                remoteDirectory: '.', remoteDirectorySDF: false, removePrefix: '', sourceFiles: 'webapp/target/webapp.war, Dockerfile, hosts, remoteplaybook.yml'
                        )]
                        , usePromotionTimestamp: false, useWorkspaceInPromotion: false, verbose: true)
                ])
        echo "Deployed on Controller node : ${controllerServer}:8080/webapp"
        echo "Deployed on QA node : ${qaServer}:8080/webapp"
    }

    stage('Cleanup'){
        cleanWs()
    }

}
