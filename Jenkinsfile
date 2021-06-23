pipeline {
        // This file is used by Jenkins to build and publish to DockerHub
        // Documentation on Jenkinsfile syntax can be located here:
        //      https://jenkins.io/solutions/pipeline/

        agent { label 'usau-dcbuild02' }
	options {
                disableConcurrentBuilds()
        }
        environment {
                registryCredential = 'avdockerhub'
                DockerImageAgent = ''
                DockerImageBotKube = ''
                DockerMonitoring = ''
                packageJSONVersion = ''
                PATH=$PATH:/usr/local/bin

                CHART_NAME=$1
                BUCKET=actian-datacloud-helm-charts
                CHART_VERSION_OLD=$(grep 'version: ' Chart.yaml | cut -f2- -d' ')
                CHART_VERSION=$(echo $CHART_VERSION_OLD | awk -F. -v OFS=. '{$NF=ENVIRON["BUILD_NUMBER"]; print $0}')
        }
        stages{
                stage('Prepare') {
                        steps {
                                script {
                                        sh "aws s3api head-object --bucket $BUCKET --key $CHART_NAME-$CHART_VERSION.tgz || not_exist=true"
                                        env.SHORTENED_BRANCH_NAME = sh returnStdout: true, script: ''' echo "${GIT_BRANCH}" | sed \'s:.*/::\' | tr -d \'\\n\' '''
                                        if (env.SHORTENED_BRANCH_NAME =~ 'master') {
                                                env.DOCKER_TAG_BN = "${env.SEMVER}.${BUILD_NUMBER}"
                                                env.DOCKER_TAG_LATEST = "latest"
                                                env.HELM_TAG = "${env.SEMVER}-${BUILD_NUMBER}"
                                        }
                                        else if (env.SHORTENED_BRANCH_NAME =~ 'release') {
                                                env.DOCKER_TAG_BN = "${env.SEMVER}"
                                                env.DOCKER_TAG_LATEST = "release"
                                                env.HELM_TAG = "${env.SEMVER}"
                                        } else {
                                                env.DOCKER_TAG_BN = "${env.SEMVER}_${env.SHORTENED_BRANCH_NAME}.${BUILD_NUMBER}"
                                                env.DOCKER_TAG_LATEST = "${env.SEMVER}_${env.SHORTENED_BRANCH_NAME}.latest"
                                                env.HELM_TAG = "${env.SEMVER}-${env.SHORTENED_BRANCH_NAME}.${BUILD_NUMBER}"
                                        }
                                }
                        }
                }
                stage ('Helm package AmqpPAS') {

                        steps {
                                script {
                                        def packageJSON = readJSON file: 'agents/kubernetes/package.json'
                                        packageJSONVersion = packageJSON.version
                                        echo packageJSONVersion
                                }
                                sh "sed -i 's/tag: master/tag: ${DOCKER_TAG_BN}/g' charts/agent/values.yaml"
                                sh "sed -i 's/appVersion: \"master\"/appVersion: \"${packageJSONVersion}\"/g' charts/agent/Chart.yaml"
                                sh "sed -i 's/version: 0.1.0/version: ${HELM_TAG}/g' charts/agent/Chart.yaml"
                                sh "sed -i 's/version: 1.0.0/version: ${HELM_TAG}/g' charts/agent/Chart.yaml"
                                sh "helm dependency update charts/agent"
                                sh "helm package --version ${HELM_TAG} charts/agent"
                                sh "aws s3api get-object --bucket ${BUCKET} --key index.yaml index-temp.yaml"
                                sh "helm repo index --merge index-temp.yaml --url https://${BUCKET}.s3.amazonaws.com/dist ."
                        }
                }
                stage('Deploy packages to S3') {
                        steps {
                                sh "aws s3api put-object --bucket ${BUCKET} --key dist/agent-${HELM_TAG}.tgz --content-type \"application/x-gzip\" --body agent-${HELM_TAG}.tgz"
                                sh "aws s3api put-object --bucket ${BUCKET} --key index.yaml --body index.yaml"
                        }
                }
                stage ('Upgrading Dev Environment'){
                         when {
                                anyOf{
                                        branch "IP-4424"
                                }
                        }
                        steps {
                                echo 'Deploy into Dev Environment'
                                sh "gcloud auth activate-service-account --key-file ${GOOGLE_SERVICE_ACCOUNT_KEY}"
                                sh "gcloud container clusters get-credentials av-gc-dev2 --region=us-east1"
                                sh "helm repo add avalanche https://${BUCKET}.s3.amazonaws.com"
                                sh "helm repo update"
                                sh "helm upgrade agent avalanche/agent -n avalanche --version=${HELM_TAG} --wait"
                        }
                }
//                 stage ('Upgrading Test Environment'){
//                          when {
//                                 anyOf{
//                                         branch "release"
//                                 }
//                         }
//                         steps {
//                                 echo 'Deploy into Test Environment'
//                                 sh "gcloud auth activate-service-account --key-file ${GOOGLE_SERVICE_ACCOUNT_KEY}"
//                                 sh "gcloud container clusters get-credentials av-gc-test --region=us-east1"
//                                 sh "helm repo add avalanche https://${BUCKET}.s3.amazonaws.com"
//                                 sh "helm repo update"
//                                 //sh "helm upgrade agent avalanche/agent -n avalanche --version=${HELM_TAG} --wait"
//                         }
//                 }
                stage('Cleanup') {
                        steps {
                                sh'''
                                        echo Cleanup
                                '''
                                }
                }
        }
        post {
                failure {
                        emailext(body: '${DEFAULT_CONTENT}', mimeType: 'text/html',
                                replyTo: 'DataConnect.Cloud.Platform@actian.com', subject: '${DEFAULT_SUBJECT}',
                                to: emailextrecipients([[$class: 'CulpritsRecipientProvider'],
                                                        [$class: 'RequesterRecipientProvider']]))
                }
        }
 }
