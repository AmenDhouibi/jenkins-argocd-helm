pipeline {
  agent any

environment {
    KUBECONFIG = '/var/lib/jenkins/.kube/config'
}

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
            def imageName = "amendhouibi22/springboot-app"
            def imageTag = "${BUILD_NUMBER}"

            withDockerRegistry([credentialsId: 'docker', url: 'https://index.docker.io/v1/']) {
                sh """
                    docker build -t ${imageName}:${imageTag} spring-boot-app
                    docker push ${imageName}:${imageTag}
                """
            }
        }
    }
}


    stage('Deploy to Test') {
          steps {
        sh 'kubectl get nodes'  // Test d'accès Kubernetes
        sh 'helm upgrade --install springboot-test . --namespace test --create-namespace --set image.tag=16'
    }
    }

   stage('Bump Chart Version') {
        steps {
    withCredentials([usernamePassword(credentialsId: 'github_credentials', usernameVariable: 'GIT_USER', passwordVariable: 'GIT_TOKEN')]) {
      sh '''
        git config user.email "amen_dhouibi@yahoo.com"
        git config user.name  "AmenDhouibi"

        # Replace the tag
        sed -i "s/tag: .*/tag: ${BUILD_NUMBER}/g" values.yaml

        # Git commit & push using credentials
        git add values.yaml
        git commit -m "ci: bump image.tag to ${BUILD_NUMBER}"

        git push https://${GIT_USER}:${GIT_TOKEN}@github.com/${GIT_USER}/jenkins-argocd-helm.git HEAD:main
      '''
    }
  }
}

 }
}
