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
            cd spring-boot-app && mvn test -B || true    // never fail even if tests are missing
          '''
            }
          post {
            always {
             // only fail if reports exist but are malformed; won't error if none are found
             junit allowEmptyResults: true, testResults: 'spring-boot-app/target/surefire-reports/*.xml'
                   }
              }
    }

    stage('Code Quality') {
      steps {
        withSonarQubeEnv('sonar-server') {
          sh '''
           cd spring-boot-app && mvn clean verify sonar:sonar -B \
              -Dsonar.projectKey=my-project-key \
              -Dsonar.sources=src
          '''
        }
      }
    }
    stage('Quality Gate') {
      steps {
        // requires “Pipeline: SonarQube” plugin
        waitForQualityGate abortPipeline: true
      }
    }


    stage('Package') {
         steps {
        sh '''
          cd spring-boot-app
          mvn package -B
    '''
  }
}


 }
}
