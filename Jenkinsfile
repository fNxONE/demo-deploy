env.DOCKERHUB_USERNAME = 'aocp'
pipeline {
  agent any
  stages {
    stage("Production") {
      try {
        // Create the service if it doesn't exist otherwise just update the image
        sh '''
          SERVICES=$(docker service ls --filter name=demo-deploy --quiet | wc -l)
          if [[ "$SERVICES" -eq 0 ]]; then
            docker network rm demo-deploy || true
            docker network create --driver overlay --attachable demo-deploy
            docker service create --replicas 3 --network demo-deploy --name demo-deploy -p 8080:8080 ${DOCKERHUB_USERNAME}/demo-deploy:${BUILD_NUMBER}
          else
            docker service update --image ${DOCKERHUB_USERNAME}/demo-deploy:${BUILD_NUMBER} demo-deploy
          fi
          '''
        
      }catch(e) {
        sh "docker service update --rollback  cd-demo"
        error "Service update failed in production"
      }finally {
        sh "docker ps -aq | xargs docker rm || true"
      }
    }
  }
}
