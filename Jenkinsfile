node {
    def app

    stage('Clone repository') {
        /* Let's make sure we have the repository cloned to our workspace */

        checkout scm
    }

    stage('Build image') {
        /* This builds the actual image; synonymous to
         * docker build on the command line */

        app = docker.build("rvarg11/helloworld")
    }

    stage('Test image') {
        /* Ideally, we would run a test framework against our image.
         * For this example, we're using a Volkswagen-type approach ;-) */

        app.inside {
            sh 'echo "Tests passed"'
        }
    }

    stage('Push image') {
        /* Finally, we'll push the image with two tags:
         * First, the incremental build number from Jenkins
         * Second, the 'latest' tag.
         * Pushing multiple tags is cheap, as all the layers are reused. */
        docker.withRegistry('https://registry.hub.docker.com', 'docker-hub-credentials') {
            app.push("latest")
            app.push("v_${env.BUILD_NUMBER}")
        }
    }

    stage('Deploy Image') {
         sh '''

         # Create a new task definition for this build
         sed -e "s;%BUILD_NUMBER%;${BUILD_NUMBER};g" docker-helloworld.json > docker-helloworld-v_${BUILD_NUMBER}.json
         aws ecs register-task-definition --family docker-helloworld --cli-input-json file://docker-helloworld-v_${BUILD_NUMBER}.json

         #update Container
         aws ecs update-service --cluster default --service sample-webapp --task-definition docker-helloworld:${BUILD_NUMBER} --desired-count 1

         '''
    }

}
