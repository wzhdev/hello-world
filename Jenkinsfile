def label = "mypod-${UUID.randomUUID().toString()}"
def image = "docker.wzh/wzhdev/hello-world"
podTemplate(label: label, cloud: 'kubernetes', yaml: """
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: maven
    image: maven:3.3.9-jdk-8-alpine
    command: ['cat']
    tty: true
    volumeMounts:
    - name: maven-repo
      mountPath: /root/.m2/repository
  - name: docker
    image: docker:18.09
    command: ['cat']
    tty: true
    volumeMounts:
    - name: dockersock
      mountPath: /var/run/docker.sock
  - name: helm
    image: wzhdev/helm
    command: ['cat']
    tty: true
  volumes:
  - name: dockersock
    hostPath:
      path: /var/run/docker.sock
  - name: maven-repo
    persistentVolumeClaim:
      claimName: maven-repo
"""
)
{
	node(label) {
		stage('Get a Maven Project') {
			git 'http://gogs-gogs/wzhdev/hello-world.git'
			container('maven') {
				sh 'mvn -B clean package'
			}
			container('docker') {
				sh "docker build -t ${image} ."
				sh "docker push ${image}"
			}
			container('helm') {
				sh "helm repo add chart https://chart.wzh"
				sh "mv charts hello-world"
				sh "cd hello-world && helm package . && helm push hello-world-0.1.0.tgz chart"
				sh "helm repo update"
				sh "helm install --name hello-world --namespace default chart/hello-world"
			}
		}
	}
}