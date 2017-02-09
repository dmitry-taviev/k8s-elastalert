node {
	slackSend("[<${env.BUILD_URL}|${env.JOB_NAME}:${env.BUILD_NUMBER}>] Pipeline started!")

	try {

		stage 'checkout'
        slackSend("[<${env.BUILD_URL}|${env.JOB_NAME}:${env.BUILD_NUMBER}>] Checking out project..")
        checkout scm
        slackSend color: 'good', message: "[<${env.BUILD_URL}|${env.JOB_NAME}:${env.BUILD_NUMBER}>] Checked out successfully. Ready for packaging!"
		
		stage 'build & push image'
        slackSend("[<${env.BUILD_URL}|${env.JOB_NAME}:${env.BUILD_NUMBER}>] Building new Docker image..")
        def service = env.JOB_NAME
        def region = 'eu-west-1'
        def repoName = "apply/smart-${service}"
        try {
            sh("aws ecr create-repository --repository-name ${repoName} --region ${region}")
        } catch (e) {}
        def dockerRepo = "944590742144.dkr.ecr.${region}.amazonaws.com/${repoName}"
        def img = docker.build("${dockerRepo}:build-${env.BUILD_NUMBER}")
        img.push()
        img.push('latest')
        slackSend color: 'good', message: "[<${env.BUILD_URL}|${env.JOB_NAME}:${env.BUILD_NUMBER}>] New image: ${img.id}"

		stage 'deploy service'
        slackSend("[<${env.BUILD_URL}|${env.JOB_NAME}:${env.BUILD_NUMBER}>] Deploying..")
        try {
            sh("kubectl delete rc --namespace=kube-system ${service}")
            sleep 30
            sh("kubectl create --namespace=kube-system -f rc.yaml")
        } catch (failure) {
            slackSend color: 'danger', message: "[<${env.BUILD_URL}|${env.JOB_NAME}:${env.BUILD_NUMBER}>] Failed to update service!"
        }

	} catch (e) {
		slackSend color: 'danger', message: "[<${env.BUILD_URL}|${env.JOB_NAME}:${env.BUILD_NUMBER}>] Pipeline failed: ${e}"
        error(e.getMessage())
	}

	slackSend color: 'good', message: "[<${env.BUILD_URL}|${env.JOB_NAME}:${env.BUILD_NUMBER}>] Pipeline finished!"
}