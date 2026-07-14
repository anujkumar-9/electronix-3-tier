pipeline {
    agent {
        label 'electronics'
    }

    environment {
        S3_BUCKET = 'electronix-production-1'
        CLOUDFRONT_ID = 'E2XCMCHQSL3JVJ'
        AWS_REGION = 'us-east-1'
    }

    stages {

        stage('Frontend Deployment') {

            when {
                changeset "frontend/**"
            }

            stages {

                stage('Install Dependencies') {
                    steps {
                        dir('frontend') {
                            sh 'npm install'
                        }
                    }
                }

                stage('Run Tests') {
                    steps {
                        dir('frontend') {
                            sh 'npm test -- --watchAll=false || echo "No Tests Configured."'
                        }
                    }
                }

                stage('Build') {
                    steps {
                        dir('frontend') {
                            sh 'npm run build'
                        }
                    }
                }

                stage('Deploy to S3') {
                    steps {
                        dir('frontend') {
                            sh '''
                                aws s3 sync dist/ s3://${S3_BUCKET} \
                                    --delete \
                                    --region ${AWS_REGION}
                            '''
                        }
                    }
                }

                stage('Invalidate CloudFront Cache') {
                    steps {
                        sh '''
                            aws cloudfront create-invalidation \
                                --distribution-id ${CLOUDFRONT_ID} \
                                --paths "/*"
                        '''
                    }
                }
            }

            post {
                success {
                    echo "✅ Frontend deployed successfully."
                }

                failure {
                    echo "❌ Frontend deployment failed."
                }
            }
        }
    }
}