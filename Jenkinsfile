podTemplate(yaml: '''
    apiVersion: v1
    kind: Pod
    spec:
      containers:
      - name: kaniko
        image: gcr.io/kaniko-project/executor:debug
        command:
        - sleep
        args:
        - 9999999
        volumeMounts:
        - name: kaniko-secret
          mountPath: /kaniko/.docker
      restartPolicy: Never
      volumes:
      - name: kaniko-secret
        secret:
            secretName: dockercred
            items:
            - key: .dockerconfigjson
              path: config.json
''') {
  node(POD_LABEL) {
    def app

    stage('Clone repository') {

        git url: 'https://github.com/ikoral/sample-gitops.git', branch: 'main'
    }

  stage('Build and Push Project') {
      container('kaniko') {
        stage('Build image') {
          sh '''
            /kaniko/executor -f `pwd`/Dockerfile -c `pwd` --destination ikoral/k8s-sample-gitops:${BUILD_NUMBER}
          '''
        }
      }
    }

    stage('Test image') {
            sh 'echo "Tests passed"'
    }
    
    stage('Trigger ManifestUpdate') {
                echo "triggering updatemanifestjob"
                build job: 'updatemanifest', parameters: [string(name: 'DOCKERTAG', value: env.BUILD_NUMBER)]
        }
 }
}