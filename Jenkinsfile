
node {
env.CLUSTER_URL = 'https://35.222.124.34/'



  stage('Clone repository') {
        def commit = checkout scm
        env.GIT_COMMIT_HASH = commit.GIT_COMMIT[0..6]   
    }



  stage('Build image') {
    
 
    
        commit_id = sh(returnStdout: true, script: 'git rev-parse --short HEAD').trim()
  
        sh 'docker-compose -f docker-compose.stage.kub.yml build'
        sh "docker tag example-app:latest qwaw1test/example-app:${commit_id}"


    }


  stage('Push image') {
        commit_id = sh(returnStdout: true, script: 'git rev-parse --short HEAD').trim()
        docker.withRegistry('https://registry.hub.docker.com', 'dockerhub') {
             docker.image("qwaw1test/example-app:${commit_id}").push()
        }

    }







timeout(time: 10, unit: 'MINUTES') {
  stage('Deploy') {
        script {
            def USER_INPUT = input(
            message: 'Сhose ENV',
            parameters: [
            [$class: 'ChoiceParameterDefinition',
            choices: ['Decline','dev','prod'].join('\n'),
            name: 'input',
            description: 'Menu - select box option']
            ])
            echo "Approve: ${USER_INPUT}"



// dev
        if( "${USER_INPUT}" == "dev"){
        commit_id = sh(returnStdout: true, script: 'git rev-parse --short HEAD').trim()
          
        withKubeConfig([credentialsId: 'secret-kube', serverUrl: "${env.CLUSTER_URL}"]) {
        sh "helm upgrade --install test-dev ./kubernetes/helm/ -f ./kubernetes/helm/values_dev.yaml --set env=dev --set appImage=qwaw1test/example-app:${commit_id}"
          }
        }

// prod
        else if ( "${USER_INPUT}" == "prod"){
        commit_id = sh(returnStdout: true, script: 'git rev-parse --short HEAD').trim()
        withKubeConfig([credentialsId: 'stagging-nl-eks-api', serverUrl: "${env.CLUSTER_URL}"]) {
        sh "helm upgrade --install test-prod ./kubernetes/helm/ --set env=prod --set appImage=qwaw1test/example-app:${commit_id}"

          }
        }




  }
}
}
}
