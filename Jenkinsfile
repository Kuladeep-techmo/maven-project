pipeline {
    agent {
        label 'Dev'
    }

    parameters {
        choice choices: ['dev', 'prod'], name: 'select_environment'
    }

    environment {
        NAME = "KULADEEP"
    }
    
    tools {
        maven 'mymaven'
    }

    stages {
        stage('build') {
            steps {
                script {
                    def file = load "script.groovy"
                    file.hello()
                }
                sh 'mvn clean package -DskipTests=true'
            }
        }

        stage('test') { 
            parallel {
                stage('testA') {
                    steps {
                        echo "This is test A"
                        sh "mvn test"
                    }
                }
                stage('testB') {
                    steps {
                        echo "This is test B"
                        sh "mvn test"
                    }
                }
            }
            post {
                success {
                    dir("webapp/target/") {
                        stash name: "maven-build", includes: "*.war"
                    }
                }
            }
        }

        stage('deploy_dev') {  
            when { 
                beforeAgent true
                expression { params.select_environment == 'dev' }
            }
            environment {
                SURNAME = "N"
            }
            steps {
                echo "HELLO ${NAME} ${SURNAME}"
                dir("/var/www/html") {
                    unstash "maven-build"
                }
                sh """
                cd /var/www/html/
                jar -xvf webapp.war
                """
            }
        }

        stage('deploy_prod') {
            agent { 
                label 'prod' 
            }
            when { 
                beforeAgent true
                expression { params.select_environment == 'prod' }
            }
            steps {
                timeout(time: 5, unit: 'DAYS') {
                    input message: 'Deployment approved?'
                }
                dir("/var/www/html") {
                    unstash "maven-build"
                }
                sh """
                cd /var/www/html/
                jar -xvf webapp.war
                """
            }  
        }

        stage('Print ASCII Banner') {
            agent { 
                label 'prod' 
            }
            steps {
                sh '''
                sudo apt-get install -y figlet || true
                figlet "DEVOPS" 
                '''
            }
        }
    }
}