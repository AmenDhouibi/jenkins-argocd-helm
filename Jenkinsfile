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
  }
}
