
pipeline {
    agent any

    options {
        timestamps()
        disableConcurrentBuilds()
        skipDefaultCheckout(true)
    }

    parameters {
        choice(
            name: 'ACTION',
            choices: ['build', 'push', 'deploy', 'rollback'],
            description: 'Choose the operation to perform'
        )
        string(
            name: 'VERSION',
            defaultValue: '',
            description: 'Release tag, for example v1.0.0'
        )
        string(
            name: 'MANAGER_HOST',
            defaultValue: '3.93.162.107',
            description: 'Public IP or DNS of the Swarm manage'
        )
    }

    environment {
        AWS_REGION = 'us-east-1'
        AWS_ACCOUNT_ID = '254209146027'
        ECR_REPOSITORY = 'burger-website'
        IMAGE_URI = '254209146027.dkr.ecr.us-east-1.amazonaws.com/burger-website'
        STACK_NAME = 'burger-website'
    }

    stages {
        stage('Validate') {
            steps {
                script {
                    env.IS_TAG_RELEASE =
                        (env.TAG_NAME?.trim()) ? 'true' : 'false'

                    if (env.IS_TAG_RELEASE == 'true') {
                        env.RELEASE_VERSION = env.TAG_NAME
                    } else {
                        env.RELEASE_VERSION = params.VERSION.trim()
                    }

                    if (!(env.RELEASE_VERSION ==~
                        /v[0-9]+\.[0-9]+\.[0-9]+/)) {
                        error('VERSION must use a tag such as v1.0.0')
                    }

                    if (params.ACTION in ['deploy', 'rollback'] ||
                        env.IS_TAG_RELEASE == 'true') {
                        if (!params.MANAGER_HOST.trim()) {
                            error('Enter MANAGER_HOST before deployment')
                        }
                    }

                    echo "Release version: ${env.RELEASE_VERSION}"
                    echo "Automatic tag release: ${env.IS_TAG_RELEASE}"
                    echo "Requested action: ${params.ACTION}"
                }
            }
        }

        stage('Checkout release source') {
            when {
                expression {
                    env.IS_TAG_RELEASE == 'true' ||
                    params.ACTION == 'build'
                }
            }
            steps {
                checkout scm
                sh '''
                    set -eu
                    git fetch --force --tags origin
                    git checkout "refs/tags/${RELEASE_VERSION}"
                    test -f Dockerfile
                    test -f docker-stack.yml
                '''
            }
        }

        stage('Build image') {
            when {
                expression {
                    env.IS_TAG_RELEASE == 'true' ||
                    params.ACTION == 'build'
                }
            }
            steps {
                sh '''
                    set -eu
                    docker build \
                      -t "${IMAGE_URI}:${RELEASE_VERSION}" .
                '''
            }
        }

        stage('Push image to ECR') {
            when {
                expression {
                    env.IS_TAG_RELEASE == 'true' ||
                    params.ACTION == 'push'
                }
            }
            steps {
                withCredentials([
                    string(credentialsId: 'aws-access-key-id',
                           variable: 'AWS_ACCESS_KEY_ID'),
                    string(credentialsId: 'aws-secret-access-key',
                           variable: 'AWS_SECRET_ACCESS_KEY')
                ]) {
                    sh '''
                        set -eu
                        export AWS_DEFAULT_REGION="$AWS_REGION"

                        if aws ecr describe-images \
                          --repository-name "$ECR_REPOSITORY" \
                          --image-ids imageTag="$RELEASE_VERSION" \
                          >/dev/null 2>&1; then
                            echo "ERROR: This release tag already exists in ECR."
                            echo "Refusing to overwrite an existing release."
                            exit 1
                        fi

                        aws ecr get-login-password \
                          --region "$AWS_REGION" |
                          docker login \
                            --username AWS \
                            --password-stdin \
                            "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com"

                        docker image inspect \
                          "${IMAGE_URI}:${RELEASE_VERSION}" >/dev/null

                        docker push "${IMAGE_URI}:${RELEASE_VERSION}"
                    '''
                }
            }
        }

        stage('Verify existing ECR image') {
            when {
                expression {
                    params.ACTION in ['deploy', 'rollback'] ||
                    env.IS_TAG_RELEASE == 'true'
                }
            }
            steps {
                withCredentials([
                    string(credentialsId: 'aws-access-key-id',
                           variable: 'AWS_ACCESS_KEY_ID'),
                    string(credentialsId: 'aws-secret-access-key',
                           variable: 'AWS_SECRET_ACCESS_KEY')
                ]) {
                    sh '''
                        set -eu
                        export AWS_DEFAULT_REGION="$AWS_REGION"

                        aws ecr describe-images \
                          --repository-name "$ECR_REPOSITORY" \
                          --image-ids imageTag="$RELEASE_VERSION" \
                          --query 'imageDetails[0].imageDigest' \
                          --output text
                    '''
                }
            }
        }

        stage('Deploy or rollback on Swarm') {
            when {
                expression {
                    params.ACTION in ['deploy', 'rollback'] ||
                    env.IS_TAG_RELEASE == 'true'
                }
            }
            steps {
                withCredentials([
                    string(credentialsId: 'aws-access-key-id',
                           variable: 'AWS_ACCESS_KEY_ID'),
                    string(credentialsId: 'aws-secret-access-key',
                           variable: 'AWS_SECRET_ACCESS_KEY')
                ]) {
                    sshagent(credentials: ['swarm-manager-ssh']) {
                        sh '''
                            set -eu
                            export AWS_DEFAULT_REGION="$AWS_REGION"

                            REGISTRY="${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com"
                            IMAGE="${IMAGE_URI}:${RELEASE_VERSION}"

                            # Confirm the target image exists before deployment.
                            aws ecr describe-images \
                              --repository-name "$ECR_REPOSITORY" \
                              --image-ids imageTag="$RELEASE_VERSION" \
                              >/dev/null

                            # Authenticate the remote manager to ECR.
                            aws ecr get-login-password \
                              --region "$AWS_REGION" |
                              ssh -o StrictHostKeyChecking=accept-new \
                                "ubuntu@${MANAGER_HOST}" \
                                "docker login --username AWS --password-stdin '${REGISTRY}'"

                            # Substitute the immutable versioned image.
                            sed "s|IMAGE_PLACEHOLDER|${IMAGE}|g" \
                              docker-stack.yml > burger-stack-rendered.yml

                            # Copy stack definition to the manager.
                            scp -o StrictHostKeyChecking=accept-new \
                              burger-stack-rendered.yml \
                              "ubuntu@${MANAGER_HOST}:/tmp/burger-stack-rendered.yml"

                            # Redeploy the existing image; rollback does not build.
                            ssh -o StrictHostKeyChecking=accept-new \
                              "ubuntu@${MANAGER_HOST}" \
                              "docker stack deploy --with-registry-auth \
                               -c /tmp/burger-stack-rendered.yml '${STACK_NAME}'"

                            # Wait for Swarm to converge and show service status.
                            ssh -o StrictHostKeyChecking=accept-new \
                              "ubuntu@${MANAGER_HOST}" \
                              "docker service ls &&
                               docker service ps '${STACK_NAME}_burger-website'"

                            echo "Deployment requested for ${IMAGE}"
                        '''
                    }
                }
            }
        }
    }

    post {
        success {
            echo 'Pipeline completed successfully.'
        }
        failure {
            echo 'Pipeline failed. Review the stage logs before retrying.'
        }
        always {
            echo 'Burger-Website release pipeline finished.'
        }
    }
}
