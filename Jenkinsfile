pipeline {

    environment {
    dockerimagename = "Nelztacy/electricity"
    dockerImage = ""
  }

  agent any
  tools{
    maven "Maven"
  }

  stages {
    stage('Checkout') {
      steps {
        checkout scm
      }
    }
    stage ('Static Application Security Testing = SAST') {
		steps {
		withSonarQubeEnv('sonar') {
			sh 'mvn sonar:sonar'
			sh 'cat target/sonar/report-task.txt'
		       }
		}
	}
  stage('Build Artifact') {
        steps {
            sh "mvn clean package"
                }
                post{
                success{
                    echo "Archiving the Artifacts"
                    archiveArtifacts artifacts: "**/target/*.jar"
                }
            }
    }
    stage('Unit test'){
        steps {
            sh "mvn test"
      }
    }
    stage ('Source-Composition-Analysis') {
	    steps {
		     sh 'rm owasp-* || true'
		     sh 'wget https://raw.githubusercontent.com/devopssecure/webapp/master/owasp-dependency-check.sh'	
		     sh 'chmod +x owasp-dependency-check.sh'
		     sh 'bash owasp-dependency-check.sh'
		     sh 'cat /var/lib/jenkins/OWASP-Dependency-Check/reports/dependency-check-report.xml'
		}
	}
  stage ('Deploy-To-Tomcat') {
        steps {
        sshagent(['solar']) {
                sh 'scp -o StrictHostKeyChecking=no target/*.jar solar@10.0.0.112:/opt/tomcat/webapps/'
              }     
           }      
    }
    stage('Copy JAR to Tomcat') {
        steps {
                script {
                    def remoteHost = '10.0.0.112'
                    def remoteUser = 'solar'
                    def remoteDir = '/opt/tomcat/webapps/'
                    def jarFileName = '*.jar'
                    def localFilePath = "/var/lib/jenkins/workspace/electricity-billing-system/target/${jarFileName}"
                    
                    sh "scp ${localFilePath} ${remoteUser}@${remoteHost}:${remoteDir}"
                }
            }
        }
        stage ('Port Scan') {
		    steps {
			sh 'rm nmap* || true'
			sh 'docker run --rm -v "$(pwd)":/data uzyexe/nmap -sS -sV -oX nmap 10.0.0.112'
			sh 'cat nmap'
		    }
	    }
      stage ('DAST') {
		    steps {
			    sshagent(['jenkins']) {
			        sh 'ssh -o StrictHostKeyChecking=no jenkins@10.0.0.112 "docker run -t owasp/zap2docker-stable zap-baseline.py -t http://10.0.0.112:8080/" || true'
			    }
			}
		}
    stage('Build docker image'){
            steps{
                script{
                    sh 'docker build -t nelzone/electricity:latest .'
                }
            }
        }
  }
}