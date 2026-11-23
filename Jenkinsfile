// CD pipeline: GitHub Actions builds and pushes the images of each commit (CI),
// Jenkins deploys exactly those images to the Kubernetes cluster.
pipeline {
  agent any
  triggers { pollSCM('H/2 * * * *') }      // check GitHub every ~2 min (no public URL for a webhook)
  options { timeout(time: 20, unit: 'MINUTES') }
  environment { REGISTRY = 'rayenbkm' }

  stages {
    stage('Version') {
      steps {
        script { env.TAG = 'sha-' + sh(script: 'git rev-parse --short=7 HEAD', returnStdout: true).trim() }
        echo "Deploying images tagged ${env.TAG}"
      }
    }

    stage('Wait for CI images') {
      steps {
        // A commit that fails to compile never gets images, so it is never deployed.
        sh '''
          for svc in backend frontend; do
            until curl -sf https://hub.docker.com/v2/repositories/$REGISTRY/devops-$svc/tags/$TAG > /dev/null; do
              echo "waiting for $REGISTRY/devops-$svc:$TAG (GitHub Actions)..."; sleep 20
            done
            echo "found $REGISTRY/devops-$svc:$TAG"
          done
        '''
      }
    }

    stage('Deploy to Kubernetes') {
      steps {
        withCredentials([file(credentialsId: 'kubeconfig', variable: 'KUBECONFIG')]) {
          sh '''
            sed -i "s/:sha-[0-9a-f]*/:$TAG/" k8s/20-backend.yaml k8s/30-frontend.yaml
            grep -h "image:" k8s/*.yaml
            kubectl apply -f k8s/
            kubectl -n fullstack rollout status deployment/backend --timeout=600s
            kubectl -n fullstack rollout status deployment/frontend --timeout=300s
          '''
        }
      }
    }

    stage('Smoke test') {
      steps {
        sh '''
          curl -sf http://fullstack-worker/ | grep -o "<title>.*</title>"
          curl -sf http://fullstack-worker/api/user/all
        '''
      }
    }
  }
}
