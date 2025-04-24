pipeline {
  agent none  // Start with no global agent, so we can define our custom node for each stage

  environment {
    REGISTRY = 'docker.io/avishilon22' // e.g. docker.io/youruser
    IMAGE_NAME_BACKEND = 'backend'
    IMAGE_NAME_FRONTEND = 'ci-frontend'
    COMMIT_SHA = "${env.GIT_COMMIT ?: 'latest'}"
  }

  stages {
    // stage('Clone Repository') {
    //   agent {
    //     kubernetes {
    //       label 'docker-agent'  // Use the Docker pod template
    //       defaultContainer 'docker'
    //       containerTemplate(name: 'docker', image: 'docker:20.10.7-dind', command: 'dockerd-entrypoint.sh', privileged: true, ttyEnabled: true)
    //     }
    //   }
    //   steps {
    //     git branch: 'main', url: 'https://github.com/avishilon26/CI-CD.git'
    //   }
    // }

    stage('Check Compilation') {
      agent {
        kubernetes {
          label 'python-agent'  // Use a separate Python testing pod template
          defaultContainer 'python'
          containerTemplate(name: 'python', image: 'python:3.9', command: 'sleep', args: 'infinity')
        }
      }
      steps {
        git branch: 'main', url: 'https://github.com/avishilon26/CI-CD.git'
        sh '''
          # Install dependencies directly in the Python pod
          pip install pytest pytest-cov
          echo "[Backend] Checking syntax..."
          python3 -m py_compile backend/*.py || exit 1
          echo "[Frontend] No compilation needed for static files"
          echo "[Backend] Running pytest..."
          cd backend
          pytest --cov=. > ../backend-coverage.txt || true
          cat ../backend-coverage.txt
          cd ..
        '''
      }
    }

    stage('Build Docker Images') {
      agent {
        kubernetes {
          label 'docker-agent'  // Switch back to Docker pod for image building
          defaultContainer 'docker'
          containerTemplate(name: 'docker', image: 'docker:dind', privileged: true, ttyEnabled: true)
        }
      }
      steps {
        git branch: 'main', url: 'https://github.com/avishilon26/CI-CD.git'
         container('docker') {
        //  sh '''
         
            
        //   # Pull the docker-compose image
        //   docker pull docker/compose:1.29.2

        //   # Run docker-compose from the pulled image
        //   docker run --rm -v /var/run/docker.sock:/var/run/docker.sock -v $PWD:/workspace -w /workspace docker/compose:1.29.2 build
        //   '''
        withCredentials([usernamePassword(credentialsId: 'docker-hub-creds', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
          sh '''
            echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
            docker build --network=host --load -t $REGISTRY/$IMAGE_NAME_BACKEND:$COMMIT_SHA ./backend
            
            docker push $REGISTRY/$IMAGE_NAME_BACKEND:$COMMIT_SHA
          '''
          }
         }

      }
    }


    stage('Deploy with Helm') {
      agent {
        kubernetes {
          label 'docker-agent'  // Use Docker pod for deployment as well
          defaultContainer 'helm'
          containerTemplate(name: 'helm', image: 'alpine/helm', command: 'sleep', args: 'infinity')
        }
      }
      steps {
        git branch: 'main', url: 'https://github.com/avishilon26/CI-CD.git'
        sh '''
          helm upgrade --install myapp .
        '''
      }
    }

    // stage('Archive Coverage Report') {
    //   steps {
    //     archiveArtifacts artifacts: 'backend-coverage.txt', allowEmptyArchive: true
    //   }
    // }
  }

}
