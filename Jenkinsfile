pipeline {
  agent any
  tools {
    maven 'Maven'
  }
  stages {
    
    stage ('Initialize') {
      steps {
        sh '''
            echo "PATH = ${PATH}"
            echo "M2_HOME = ${M2_HOME}"
           '''
      }
    }
   
    stage ('Check-Git-secrets') {
      steps {
        sh 'rm trufflehog || true'
        sh 'docker run gesellix/trufflehog --json https://github.com/prayag5198/webapp.git > trufflehog'
        sh 'cat trufflehog'
      }
    }
    
   stage ('Source-Composition-Analysis') {
      steps {
        sh 'rm owasp* || true'
        sh 'wget "https://raw.githubusercontent.com/prayag5198/webapp/master/owasp-dependency-check.sh" '
        sh 'chmod +x owasp-dependency-check.sh'
        sh 'bash owasp-dependency-check.sh'
        sh 'cat /var/lib/jenkins/OWASP-Dependency-Check/reports/dependency-check-report.xml'
      }
    }
    
    stage ('SAST') {
      steps {
        withSonarQubeEnv('sonar') {
          sh 'mvn sonar:sonar'
          sh 'cat target/sonar/report-task.txt'
        }
      }
    }
    
    stage ('Build') {
      steps {
        sh 'mvn clean package'
      }
    }
    
    stage('Deploy') {
      steps {
        deploy adapters: [tomcat9(credentialsId: '7d96ddda-581c-45b0-946f-ea885bf11c32', path: '', url: 'http://localhost:8000/')], contextPath: 'devsecops', war: '**/*.war'
        }
    }
    stage('DAST') {
      steps {
        sh 'docker run -t owasp/zap2docker-stable zap-baseline.py -t http://localhost:8000/devsecops/'
      }
    }
  }
}
