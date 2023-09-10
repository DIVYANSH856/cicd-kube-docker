pipeline {
    agent any

    environment {
        registry = "divyanshkohli856/vprofileappdock"
        registryCredentials = 'dockerhub'
    }

    stages {
        stage('BUILD') {
            steps {
                sh 'mvn clean install -DskipTests'
            }
            post {
                success {
                    echo 'Now Archiving...'
                    archiveArtifacts artifacts: '**/target/*.war'
                }
            }
        }

        stage('UNIT TEST') {
            steps {
                sh 'mvn test'
            }
        }

        stage('INTEGRATION TEST') {
            steps {
                sh 'mvn verify -DskipTests'
            }
        }

        stage('CODE ANALYSIS WITH CHECKSTYLE') {
            steps {
                sh 'mvn checkstyle:checkstyle'
            }
            post {
                success {
                    echo 'Generated Analysis Result'
                }
            }
        }

        stage('CODE ANALYSIS with SONARQUBE') {
            environment {
                scannerHome = tool 'sonar4.4.0.2170'
            }

            steps {
                withSonarQubeEnv('sonar-pro') {
                    sh """${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=vprofile \
                       -Dsonar.projectName=vprofile-repo \
                       -Dsonar.projectVersion=1.0 \
                       -Dsonar.sources=src/ \
                       -Dsonar.java.binaries=target/test-classes/com/visualpathit/account/controllerTest/ \
                       -Dsonar.junit.reportsPath=target/surefire-reports/ \
                       -Dsonar.jacoco.reportsPath=target/jacoco.exec \
                       -Dsonar.java.checkstyle.reportPaths=target/checkstyle-result.xml"""
                }

                timeout(time: 5, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Build App Image') {
            steps {
                script {
                    dockerImage = docker.build("${registry}:V${BUILD_NUMBER}")
                }
            }
        }

        stage('Upload Image') {
            steps {
                script {
                    docker.withRegistry('', registryCredentials) {
                        dockerImage.push()
                        dockerImage.push('latest')
                    }
                }
            }
        }

        stage('Remove Unused docker image') {
            steps {
                sh "docker rmi ${registry}:V${BUILD_NUMBER}"
            }
        }

        stage('Kubernetes Deploy') {
            agent { label 'KOPS' }
            steps {
                // sh "kops create cluster --name=kubevpro.learningdevops1008.software --state=s3://vprofile-kops-state11 --zones=us-east-1a,us-east-1b --node-count=2 --node-size=t2.micro --master-size=t2.medium --dns-zone=kubevpro.learningdevops1008.software --node-volume-size=8 --master-volume-size=8"
                // sh "kops update cluster --name=kubevpro.learningdevops1008.software --yes --admin --state=s3://vprofile-kops-state11"
                // timeout(time: 7, unit: 'MINUTES') {
                //     sh "kops validate cluster --name=kubevpro.learningdevops1008.software --state=s3://vprofile-kops-state11 --wait 7m"
                // }
                // sh "kubectl create namespace prod"
                sh "helm repo update"
                sh "helm upgrade --install --force vprofile-stack helm/vprofilechart --set appimage=${registry}:V${BUILD_NUMBER} --namespace prod"
            }
        }
    }
}
