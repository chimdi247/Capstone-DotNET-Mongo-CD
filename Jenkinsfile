pipeline {
    agent any

    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'main', credentialsId: 'git-token', url: 'https://github.com/chimdi247/Capstone-DotNET-Mongo-CD.git'
            }
        }

        stage('Deploy To Kubernetes') {
            steps {
                script {
                    withKubeConfig(caCertificate: '', clusterName: 'devopsshack-cluster', contextName: '', credentialsId: 'k8s-token', namespace: 'webapps', restrictKubeConfigAccess: false, serverUrl: 'https://CF8B4F6427CDAF3A4A8ADD32CBF02203.gr7.eu-west-2.eks.amazonaws.com') {
                        sh 'kubectl apply -f Manifest/manifest.yaml -n webapps'
                        sh 'kubectl apply -f Manifest/ci.yaml'
                        sh 'kubectl apply -f Manifest/ingress.yaml -n webapps'
                        sleep 30
                    }
                }
            }
        }

         stage('Verify The Deployment') {
            steps {
                script {
                    withKubeConfig(caCertificate: '', clusterName: 'devopsshack-cluster', contextName: '', credentialsId: 'k8s-token', namespace: 'webapps', restrictKubeConfigAccess: false, serverUrl: 'https://CF8B4F6427CDAF3A4A8ADD32CBF02203.gr7.eu-west-2.eks.amazonaws.com') {
                        sh 'kubectl get pods -n webapps'
                        sh 'kubectl get svc -n webapps'
                        sh 'kubectl get ingress -n webapps'
                        sleep 30
                    }
                }
            }
        }

    }
    post {
    always {
        script {
            def jobName = env.JOB_NAME
            def buildNumber = env.BUILD_NUMBER
            def pipelineStatus = currentBuild.result ?: 'UNKNOWN'
            def bannerColor = pipelineStatus.toUpperCase() == 'SUCCESS' ? 'green' : 'red'

            def body = """
                <html>
                <body>
                <div style="border: 4px solid ${bannerColor}; padding: 10px;">
                <h2>${jobName} - Build ${buildNumber}</h2>
                <div style="background-color: ${bannerColor}; padding: 10px;">
                <h3 style="color: white;">Pipeline Status: ${pipelineStatus.toUpperCase()}</h3>
                </div>
                <p>Check the <a href="${BUILD_URL}">console output</a>.</p>
                </div>
                </body>
                </html>
            """

            emailext (
                subject: "${jobName} - Build ${buildNumber} - ${pipelineStatus.toUpperCase()}",
                body: body,
                to: 'shimdi247@gmail.com',
                from: 'chimdi247@gmail.com',
                replyTo: 'jenkins@devopsshack.com',
                mimeType: 'text/html',

            )
        }
    }
}
}
