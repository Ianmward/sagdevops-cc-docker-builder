podTemplate(
    label: 'mypod', 
    inheritFrom: 'default',
    containers: [
        containerTemplate(
            name: 'docker', 
            image: 'docker:18.06',
            ttyEnabled: true,
            command: 'cat'
        )
    ],
	envVars: [
		secretEnvVar(key: 'DOCKER_USR', secretName: 'docker-store-cred', secretKey: 'username'),
		secretEnvVar(key: 'DOCKER_PSW', secretName: 'docker-store-cred', secretKey: 'password'),
		secretEnvVar(key: 'NEXUS_USR', secretName: 'docker-nexus-cred', secretKey: 'username'),
		secretEnvVar(key: 'NEXUS_PSW', secretName: 'docker-nexus-cred', secretKey: 'password'),
		secretEnvVar(key: 'EMPOWER_USR', secretName: 'empower-cred', secretKey: 'username'),
		secretEnvVar(key: 'EMPOWER_PSW', secretName: 'empower-cred', secretKey: 'password'),
		envVar(key: 'COMPOSE_PROJECT_NAME', value: 'sagdevopsccdockerbuilder'),
		envVar(key: 'RELEASE', value: '10.1'),
	],
    volumes: [
        hostPathVolume(
            hostPath: '/var/run/docker.sock',
            mountPath: '/var/run/docker.sock'
        )
    ]
) {
    node('mypod') {
        def commitId
        stage ('Extract') {
            checkout scm
            commitId = sh(script: 'git rev-parse --short HEAD', returnStdout: true).trim()
        }
        def repository
        stage ('Docker') {
            container('docker') {
                sh "docker login -u ${env.DOCKER_USR} -p ${env.DOCKER_PSW}"
            }
        }
        stage("Build") {
            container('docker') {
                sh 'cp init-${RELEASE}-dev.yaml init.yaml && cat init.yaml'
                sh 'docker -v'
                sh 'docker build -f Dockerfile.simple -t simple/msc:$RELEASE --build-arg RELEASE=$RELEASE .'
                sh 'docker build -f Dockerfile.unmanaged -t unmanaged/msc:$RELEASE --build-arg RELEASE=$RELEASE .'
                sh 'docker build -f Dockerfile.managed -t managed/msc:$RELEASE --build-arg RELEASE=$RELEASE .'
                sh 'docker images | grep msc'
            }
        }
        stage("Test") {
            container('docker') {
                sh 'docker-compose run --rm init'
                sh 'docker-compose up -d managed'
                sh 'docker-compose run --rm test'
            }
            post {
                always {
                    sh 'docker-compose logs'
                    sh 'docker-compose down'
                }
            }
        }
    }
}
