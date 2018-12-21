def label = "mypod-${UUID.randomUUID().toString()}"
def image = "harbor.tst.xiaowu.com/xiaowu/hello-world"
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
			git 'https://github.com/wzhdev/hello-world.git'
			container('maven') {
				sh 'mvn -B clean package'
			}
			container('docker') {
				sh "docker build -t ${image} ."
				sh "docker login harbor.wzh -p 123456 -u admin"
				sh "docker push ${image}"
			}
		}
	}
}