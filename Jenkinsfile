pipeline {
    agent {
        label "helm"
    }

    triggers {
        pollSCM '@hourly'
    }

    parameters {
        string defaultValue: 'none', description: 'Version to deploy', name: 'VERSION', trim: true
    }

    options {
        ansiColor('xterm')
        buildDiscarder logRotator(artifactDaysToKeepStr: '', artifactNumToKeepStr: '', daysToKeepStr: '', numToKeepStr: '15')
        timestamps()
        disableConcurrentBuilds()
    }

    stages {
        stage('Deploy to k8s') {

            when {
                allOf {
                    expression { return params.VERSION != 'none' }
                    expression { return params.VERSION != '' }
                }
            }

            steps {
                withKubeConfig(credentialsId: 'a9fe556b-01b0-4354-9a65-616baccf9cac') {
                    sh "helm upgrade -n funnel -i --set image.tag=${params.VERSION} chronos helm/chronos"
                    sh "helm upgrade -n funnel -i --set image.tag=${params.VERSION} notifier helm/notifier"
                    sh "helm upgrade -n funnel -i --set image.tag=${params.VERSION} persistence helm/persistence"
                    sh "helm upgrade -n funnel -i --set image.tag=${params.VERSION} rest helm/rest"
                    sh "helm upgrade -n funnel -i --set image.tag=${params.VERSION} retriever helm/retriever"
                }
            }
        }
    }

    post {
        always {
            mail to: "rafi@guengel.ch",
                    subject: "${JOB_NAME} (${env.BUILD_DISPLAY_NAME}) -- ${currentBuild.currentResult}",
                    body: "Refer to ${currentBuild.absoluteUrl}"
        }
    }
}
