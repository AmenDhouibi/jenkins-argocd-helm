pipeline {
  agent any

  stages {
    stage('Checkout') {
      steps {
        // pull your Git repo
        checkout scm
      }
    }
    stage('Build') {
   steps {
     sh 'cd spring-boot-app && mvn clean compile -B'
         }
    }
  stage('Unit Tests') {
     steps {
          sh '''
     cd spring-boot-app &&  mvn test -B
    '''
    junit '**/target/surefire-reports/*.xml'
  }
}

  }
}
