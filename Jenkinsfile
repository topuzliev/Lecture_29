def appName="my-app"
def appVersion="1.0-SNAPSHOT"

podTemplate(cloud: 'kubernetes', containers: [
    containerTemplate(
        name: 'jnlp',
        image: 'topuzliev/terraform:latest',
        ttyEnabled: true,
        privileged: false,
        alwaysPullImage: false,
        workingDir: '/home/jenkins/agent',
        resourceRequestCpu: '500m',
        resourceLimitCpu: '500m',
        resourceRequestMemory: '1024Mi',
        resourceLimitMemory: '1024Mi',
        envVars: [
            envVar(key: 'JENKINS_URL', value: 'http://jenkins.jenkins.svc.cluster.local:8080'),
        ]
    ),
]
)
{
        node(POD_LABEL){
            container('jnlp') {
                tool name: 'maven', type: 'maven'
                stage('Check prerequests'){
                withEnv(["PATH=${env.PATH}:${tool 'maven'}/bin"]){
                sh 'env | grep PATH'
                sh 'mvn -v'
                }
            }  

            stage('Get sources'){
                git(url: 'https://github.com/jenkins-docs/simple-java-maven-app.git', branch: "master")
            }

            stage('Build'){
                withEnv(["PATH=${env.PATH}:${tool 'maven'}/bin"]){
                    sh 'mvn -B -DskipTests clean package'
                    }        
            }
            stage('Test'){
                withEnv(["PATH=${env.PATH}:${tool 'maven'}/bin"]){
                    sh 'mvn test'
                    stash includes: 'target/my-app-1.0-SNAPSHOT.jar', name: 'artifactStash'
                }        
            }
        }
    }
}

podTemplate(cloud: 'kubernetes', containers: [
    containerTemplate(
        name: 'docker',
        image: 'docker:19.03.1-dind',
        ttyEnabled: true,
        privileged: true,
        alwaysPullImage: false,
        workingDir: '/home/jenkins/agent',
        resourceRequestCpu: '400m',
        resourceLimitCpu: '400m',
        resourceRequestMemory: '1024Mi',
        resourceLimitMemory: '1024Mi',
        envVars: [
            envVar(key: 'JENKINS_URL', value: 'http://jenkins-master:8080'),
        ]
    ),
]
)
{
        node(POD_LABEL){
            container('docker') {

            stage('Ckeck prerequest'){
                sh 'docker -v'
                sh 'ls -l /var/run/docker.sock'
            } 

            stage('Get Dockerfile'){
                git(url: 'https://github.com/topuzliev/Lecture_22_25.02.2020.git', branch: "master")
            }

            stage('Unstash application'){
                unstash 'artifactStash'
            }

            stage('Build Dockerfile'){
                dockerImage = docker.build("topuzliev/myappdocker:latest", "--no-cache --build-arg APP_NAME=${appName} --build-arg APP_VERSION=${appVersion} .")
            }
                
            stage('Push Image'){
                withDockerRegistry(credentialsId: 'dockerhub', url: 'https://index.docker.io/v1/'){
                    sh "docker push topuzliev/myappdocker:latest"
                }
            }
        }
    }
}