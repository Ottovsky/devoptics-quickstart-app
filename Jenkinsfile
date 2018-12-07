def label = UUID.randomUUID().toString()

timestamps {

  podTemplate(label: label, yaml: """
apiVersion: v1
kind: Pod
spec:
  serviceAccountName: jenkins
  containers:
  - name: maven
    command:
    - cat
    tty: true
    env:
    - name: MAVEN_OPTS
      value: -Dorg.slf4j.simpleLogger.log.org.apache.maven.cli.transfer.Slf4jMavenTransferListener=warn
    - name: BUILD_NUMBER
      value: ${env.BUILD_NUMBER}
    - name: BRANCH_NAME
      value: ${env.BRANCH_NAME}
    - name: _JAVA_OPTIONS
      value: -Xmx300M
    image: maven:3.5.4-jdk-8
    resources:
      limits:
        memory: 1500Mi
      requests:
        cpu: 100m
        memory: 1500Mi
  - name: docker
    command:
    - cat
    tty: true
    image: docker:18.09
    env:
    - name: BUILD_NUMBER
      value: ${env.BUILD_NUMBER}
    - name: BRANCH_NAME
      value: ${env.BRANCH_NAME}
    volumeMounts:
    - name: 'docker-socket'
      mountPath: /var/run/docker.sock
  volumes:
  - name: 'docker-socket'
    hostPath:
     path: /var/run/docker.sock
     type: File
    """) {
    node(label) {
      stage('Checkout') {
        checkout scm
      }
      stage('Package') {
        try {
          container('maven') {
            sh 'mvn -B clean install -Dmaven.test.skip=true'
          }
          container('docker'){
            def dockerImage = docker.build("ottovsky/devoptics-quickstart-app/devoptics-quickstart-app:${env.BUILD_ID}")
            dockerImage.push()
          }
            
        } finally {
          archiveArtifacts allowEmptyArchive: true, artifacts: '**/target/*.hpi,**/target/*.jpi'
        }
      }
    }
  }

}
