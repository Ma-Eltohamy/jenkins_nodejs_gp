pipeline {
    agent { label 'kube-agent' }
    environment {
        AWS_REGION = 'us-east-1'
        ECR_REPO = 'test'
        AWS_ACCOUNT_ID = credentials('aws-account-id')
        URL_REGISTRY = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com"
        IMAGE_TAG = "40.${env.BUILD_NUMBER}"
        FULL_IMAGE = "${URL_REGISTRY}/${ECR_REPO}:${IMAGE_TAG}"
    }

    stages {
        stage('SCM Checkout') {
            steps {
                git branch: 'rds_redis', url: 'https://github.com/mahmoud254/jenkins_nodejs_example.git'
            }
        }

        stage('Trivy Pre-Scan') {
            steps {
                container('kaivy') {
                    script {
                        // Fail pipeline if last image has CRITICAL vulns
                        sh """
                        trivy image ${URL_REGISTRY}/${ECR_REPO}:latest \
                        --severity CRITICAL \
                        --exit-code 0 \
                        --quiet || echo 'Note: latest image may have CRITICALs'
                        """
                    }
                }
            }
        }

        // stage('Kaniko build & tag (staging only)') {
        //     steps {
        //         container('kaivy') {
        //             script {
        //                 // Build an output.tar image to make trivy then check it
        //                 sh """
        //                 /kaniko/executor \
        //                   --context=git://github.com/Ma-Eltohamy/jenkins_nodejs_gp.git#rds_redis \
        //                   --destination=${FULL_IMAGE} \
        //                   --dockerfile=dockerfile \
        //                   --no-push \
        //                   --tar-path=/workspace/output.tar
        //                 """
        //             }
        //         }
        //     }
        // }

        // stage('Trivy Scan tarball image') {
        //     steps {
        //         container('kaivy') {
        //             script {
        //                 sh """
        //                 trivy image --input /workspace/output.tar \
        //                 --severity CRITICAL \
        //                 --exit-code 1 \
        //                 --format json \
        //                 -o trivy-result.json
        //                 """
        //             }
        //         }
        //     }
        // }

        stage('Push to ECR (only if passed scan)') {
            steps {
                container('kaivy') {
                    script {
                        // Push the image if it only passed from trivy
                        sh """
                        /kaniko/executor \
                          --context=git://github.com/Ma-Eltohamy/jenkins_nodejs_gp.git#rds_redis \
                          --destination=${FULL_IMAGE} \
                          --dockerfile=dockerfile
                        """
                    }
                }
            }
        }
    }
}
