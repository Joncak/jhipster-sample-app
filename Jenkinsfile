#!groovy
podTemplate( label: 'java', containers: [
    containerTemplate( name: 'docker', image: 'docker:1.11', ttyEnabled: true, command: 'cat' ),
    containerTemplate( name: 'kubectl', image: 'smesch/kubectl', ttyEnabled: true, command: 'cat' ),
    containerTemplate( name: 'maven', image: 'openjdk:8', ttyEnabled: true, command: 'cat' )
], volumes: [
    persistentVolumeClaim( mountPath: '/root/.m2/repository', claimName: 'maven-repo', readOnly: false ),
    hostPathVolume( hostPath: '/var/run/docker.sock', mountPath: '/var/run/docker.sock' ),
    secretVolume( secretName: 'kube-config', mountPath: '/root/.kube' )]
) {
    def image = 'joncak/appdemo-app'
//    def gitUrl = 'https://github.com/Joncak/jhipster-sample-app.git'

    node( 'java' ) {
//    git url: "${gitUrl}"
        checkout scm
        stage( 'Unit test' ) {
            container( 'maven' ) {
                sh 'chmod +x mvnw'
                sh './mvnw clean test'
            }
        }
        stage( 'build and push' ) {
            container( 'maven' ) {
                sh './mvnw package -DskipTests'
            }
            container( 'docker' ) {
                sh "docker build -t ${image}:latest ."
            }
        }
        container( 'kubectl' ) {
            stage( 'Deploy New Build To Kubernetes' ) {
                sh( "kubectl apply -f k8s/" )
            }
        }
    }
}
