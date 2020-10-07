node {
  env.DOCKER_HOME = "${tool name: 'docker', type: 'org.jenkinsci.plugins.docker.commons.tools.DockerTool'}"
  env.PATH="${env.DOCKER_HOME}/bin:${env.PATH}"
  env.JAVA_HOME="/usr/lib/jvm/jdk1.8.0_231"
  env.http_proxy="http://www-proxy.us.oracle.com:80/"
  env.https_proxy= "http://www-proxy.us.oracle.com:80/"
  env.exm_dev='context-cztmyldha4t'
  env.ocir='https://us-phoenix-1.ocir.io'
  env.ocir_path="us-phoenix-1.ocir.io/ax74vd5jywob/financials"
  env.JDEV_HOME = "/var/lib/jenkins/shared_files/jdevadf/oracle"

  env.dockertag="1.0.${BUILD_NUMBER}"
 
  println env.PATH
 
  try {
    stage('Checkout') {
      /* Checking Out the branch, using 'GIT_REPO_BRANCH' build parameter */
      git branch: "master", url: 'https://github.com/DeviSnigdhaM/helloWorld-helidon.git'
      /**
       * Print environment variables
       */
      sh '''
        printenv
      '''
    }
    stage('Unit Tests') {
      /**
      * Run Unit tests
      */
      withMaven (
      maven : 'maven-3.6.1',
      globalMavenSettingsConfig: 'ad48980e-3d53-4a03-aa60-24138cbfeeed') {
        sh '''
          cd $WORKSPACE/helidon-quickstart-se
          mvn test
        '''
      }
    }
    stage('Build') {
      /**
      * Build app and generate the dockerfile from template
      */
      withMaven (
      maven : 'maven-3.6.1',
      globalMavenSettingsConfig: 'ad48980e-3d53-4a03-aa60-24138cbfeeed') {
        sh '''
          cd $WORKSPACE/helidon-quickstart-se
          mvn package
        '''
      }
    }
    stage('Dockerize') {
      /**
      * Publish to OCIR
      */
      sh '''
          echo "Build docker Image"
          cd $WORKSPACE/helidon-quickstart-se
          docker build -t app:1 .
          docker tag app:1 ${ocir_path}/helloWorld:${dockertag}
      '''
      script {
          docker.withRegistry( "${ocir}", "devi_spectra_oci" ) {
             sh '''
              docker push ${ocir_path}/helloWorld:${dockertag}
            '''
          }
        }
    }
    stage('Deploy') {
      /**
      * Deploy to Kubernetes
      */
      sh '''
        echo "Publishing to OKE using kubectl"
        cd $WORKSPACE/helidon-quickstart-se
        kubectl config use-context ${exm_dev}
        sed -i -E "s/image_tag/${dockertag}/g" app.yaml
        kubectl apply -f app.yaml
      '''
    }
  } catch (err) {
    print "${err}"
    currentBuild.result = "FAILURE"
  } finally {
    stage('Post Build steps') {
      sh '''
        echo 'Post build steps go here'
      '''
    }
  }
}