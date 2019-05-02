
pipeline {
    agent any

    environment {
        BUILD_ID_UNIQUE = sh (
            script: '(echo $BRANCH_NAME$BUILD_NUMBER | md5sum | cut -f 1 -d " ")',
            returnStdout: true
        ).trim()

        CONFIGURATIONS = sh (
            returnStdout: true,
            script: 'echo $(find apps/ -maxdepth 1 -type d | sort | grep "apps/[a-zA-Z]")" conf scripts tasks"'
        )
    }

    stages {
        stage('Building Docker Image') {
            steps {
                sh 'printenv'
                sh 'docker build --no-cache -t localhost:5151/bees-web:$BUILD_ID_UNIQUE .'
            }
        }
        // stage('Create the Docker Compose file') {
        //     steps {
        //         // sh 'cp docker-compose.yml ~/workspace/pv_test_matrix/b_'+BUILD_ID_UNIQUE+'-docker-compose.yml'
        //         // sh 'cp docker-compose.yml ~/workspace/pv_apitest_matrix_AWS/b_api_'+BUILD_ID_UNIQUE+'-docker-compose.yml'

        //         // This stuff should all change to a k8s pod
        //     }
        // }
        stage("Parallel Push to Registries") {
            steps {
                parallel (
                    "push to CMB-Registry" : {
                        sh 'docker push localhost:5151/bees-web:'+BUILD_ID_UNIQUE+' || true'
                    }
                )
            }
        }

        stage('Testing') {
            steps {
                parallel (
                    "CI tests" : {
                        build job: 'pv_test_matrix_aws', parameters: [[$class: 'StringParameterValue', name: 'build', value: BUILD_ID_UNIQUE], [$class: 'StringParameterValue', name: 'commit', value: GIT_COMMIT], [$class: 'StringParameterValue', name: 'branch', value: BRANCH_NAME], [$class: 'StringParameterValue', name: 'configurations', value: CONFIGURATIONS]]
                    }
                    // ,
                    // "API tests" : {
                    // 	build job: 'pv_apitest_matrix_aws', parameters: [[$class: 'StringParameterValue', name: 'build', value: BUILD_ID_UNIQUE], [$class: 'StringParameterValue', name: 'commit', value: GIT_COMMIT], [$class: 'StringParameterValue', name: 'branch', value: BRANCH_NAME]]
                    // },
                    // "Fab Check" : {
                    //     build job: 'pv_fabcheck_aws', parameters: [[$class: 'StringParameterValue', name: 'build', value: BUILD_ID_UNIQUE], [$class: 'StringParameterValue', name: 'commit', value: GIT_COMMIT]]
                    // },
                    // "DataCommon Dependency Check": {
                    //     sh 'testing/data-common-dep-check.sh'
                    // }
                    // Pylint takes too damn long to run on each commit!
                    // },
                    // "Pylint" : {
                    // build job: 'pv_pylint_aws', parameters: [[$class: 'StringParameterValue', name: 'build', value: BUILD_ID_UNIQUE]]
                    // }
                )
            }
        }
    }

    // post {
    //     success {
    //         slackSend color: "#00BF2D", channel: "#server-alerts", message: ":pill: Project Vitamin:$BRANCH_NAME passed tests (aws). (<${env.BUILD_URL}console|Console>)"
    //     }
    //     failure {
    //         slackSend color: "#BA0028", channel: "#server-alerts", message: ":pill: Project Vitamin:$BRANCH_NAME failed tests (aws). (<${env.BUILD_URL}|status>) (<${env.BUILD_URL}console|Console>)"
    //     }
    // }
}

