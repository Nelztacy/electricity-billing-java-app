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
  }
}