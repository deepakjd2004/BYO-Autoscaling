def linodeId = ""
def linodeStatus = ""
pipeline {
    agent any

    environment {
        LINODE_CLI_TOKEN = credentials('linode-cli-token')
        ROOT_PASS = credentials('linode-root-pass')
    }

    stages {
        stage('Infrastructure Check') {
            steps {
                sh "linode-cli linodes list"
            }
        }

        stage('Provision') {
            steps {
                script {
                    linodeId = sh(
                        script: "linode-cli linodes create --root_pass $ROOT_PASS --region ap-south --image linode/debian10 --type g6-nanode-1 --json",
                        returnStdout: true
                    ).trim()

                    // Extract the Linode ID from the JSON response
                    linodeId = linodeId.replaceAll('\\[|\\]', '').split(',')[0].split(':')[1].trim()

                    echo "Linode ID: ${linodeId}"

                    // Wait for Linode creation to complete
                    timeout(time: 2, unit: 'MINUTES') {
                     //   def linodeStatus = ""

                        // Loop until the Linode is running or the timeout is reached
                        while (linodeStatus != '[running]') {
                            def linodeInfo = sh(
                                script: "linode-cli linodes view ${linodeId} --json",
                                returnStdout: true
                            ).trim()

                            // Parse the JSON response and extract the status field
                            echo "${linodeInfo}"
                            def statusField = readJSON(text: linodeInfo).status
                            echo "${statusField}"
                            linodeStatus = statusField ? statusField.toString().toLowerCase() : ""
                            echo "${linodeStatus}"

                            if (linodeStatus != '[running]') {
                                sleep time: 10, unit: 'SECONDS'
                            }
                        }
                    }

                    echo 'Linode creation completed'
                }
            }
        }

               stage('Verify') {
            steps {
                script {
                    // Verify if the Linode exists and is in running state
                  //  def linodeStatus = sh (
                //        script: "linode-cli linodes view ${linodeId} --json",
                 //       returnStdout: true
                  //  ).trim()

                    // Extract the status from the JSON response
                    //linodeStatus = sh (
                    //    script: "${linodeStatus} | jq -r '.[0].status'",
                    //    returnStdout: true
                    //).trim()

                    if (linodeStatus == '[running]') {
                        echo 'Linode is running, verification successful'
                    } else {
                        error 'Linode is not running, verification failed'
                    }
                }
            }
        }

        stage('Deprovision') {
            steps {
                sh "linode-cli linodes delete ${linodeId}"
            }
        }
    }
}

