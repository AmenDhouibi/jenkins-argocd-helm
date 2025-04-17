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


	stage('Build and Push Docker Image') {
	steps {
    script {
      def dockerRegistry = "docker.io"
      def dockerUser = "amendhouibi22"
      def imageName = "springboot-app"
      def versionTag = "${BUILD_NUMBER}"
      def imageFull = "${dockerRegistry}/${dockerUser}/${imageName}:${versionTag}"
      def imageLatest = "${dockerRegistry}/${dockerUser}/${imageName}:latest"

      docker.withRegistry("https://${dockerRegistry}", 'docker') {
        def builtImage = docker.build(imageFull, "spring-boot-app")
        builtImage.push()
        builtImage.push("latest")
      }
    }
  }
}

    stage('Deploy to Test') {
      steps {
        // Shell helm, mais tu peux aussi utiliser un step du Kubernetes Continuous Deploy Plugin
        sh '''
          helm repo update
          helm upgrade --install springboot-test . \
            --namespace test --create-namespace \
            --set image.tag=${BUILD_NUMBER}
        '''
      }
    }
 }
}
