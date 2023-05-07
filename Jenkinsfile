pipeline {
    agent any
    environment {
        registryCrendential = 'ecr:ap-south-1:awscred'
        ecrRegistry = '543684560202.dkr.ecr.ap-south-1.amazonaws.com/ci-docker-ecr' // ecr registry uri
        registryUrl = 'https://543684560202.dkr.ecr.ap-south-1.amazonaws.com' // ecr registry url
    }

    stages {
        stage('FETCH CODE') {
          steps {
            git branch: 'docker', url: 'https://github.com/devopshydclub/vprofile-project.git'
          }
        }

        stage('mvn TEST') {
            steps {
                sh 'mvn test'
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

        stage('SonarQube analysis') {
            environment {
              scannerHome = tool 'sonarqube scanner' // the name you have given the Sonar Scanner (in Global Tool Configuration)
            }
            steps { // the name you have given the Sonar Server (in configure system)
                withSonarQubeEnv(installationName: 'sonarqube server') {
                   sh '''${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=ci-code-report \
                   -Dsonar.projectName=ci-code-report-jenkins \
                   -Dsonar.projectVersion=1.0 \
                   -Dsonar.sources=src/ \
                   -Dsonar.java.binaries=target/test-classes/com/visualpathit/account/controllerTest/ \
                   -Dsonar.junit.reportsPath=target/surefire-reports/ \
                   -Dsonar.jacoco.reportsPath=target/jacoco.exec \
                   -Dsonar.java.checkstyle.reportPaths=target/checkstyle-result.xml'''
                }
            }
        }

        stage('Quality Gates') {
            steps {
                timeout(time: 1, unit: 'HOURS') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('BUILD IMAGE') {
            steps {
                script {
                    dockerImage = docker.build( ecrRegistry + ':$BUILD_NUMBER', './Docker-files/app/multi-stage/')
                }
            }
        }

        stage('UPLOAD IMAGE TO ECR') {
            steps {
                script {
                    docker.withRegistry( registryUrl, registryCrendential) {
                        dockerImage.push('$BUILD_NUMBER')
                        dockerImage.push('latest')
                    }
                }
            }
        }
   
    }

}