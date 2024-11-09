gitUrl = 'https://github.com/kimuraiivan/MobEAD'
branch = 'main'
userCred=''
node{
	stage ('Git Checkout') {
		checkout([
		$class: 'GitSCM',
		branches: [[name: '*/'+ branch ]],
		doGenerateSubmoduleConfigurations: false,
		extensions: [],
		submoduleCfg: [],
		userRemoteConfigs: [[credentialsId: userCred, url: gitUrl]]])
	}
    stage('SonarQube analysis') {
        
		env.NODEJS_HOME = "${tool 'node'}"
		env.PATH="${env.NODEJS_HOME}/bin:${env.PATH}"
		sh "node -v"
		def scannerHome = tool 'sonarqubescanner6.2.1.4610'; 
        withSonarQubeEnv('sonarQube') { 
          sh "${scannerHome}/bin/sonar-scanner  \
          -Dsonar.projectKey=jenkinsproject \
          -Dsonar.sources=. \
          -Dsonar.host.url=http://10.0.20.101:9000 \
          -Dsonar.token=sqp_cbf309e515f98d0f66e68aa0877401cb28c1571f"
        }
    }
	 stage('Build'){
		sh """
            docker build -t ivankimura/unyleya:$BUILD_NUMBER -f Dockerfile .
        """
    }
    stage('Deploy Dev'){
        try{
			sh "docker rm -f ivankimura_app_dev"
		} catch (Exception  e) {
			sh "echo $e"
		}
		sh """
            echo "Deploy dev"
			docker run -d -p 80:80 --name=ivankimura_app_dev ivankimura/unyleya:$BUILD_NUMBER
        """
    }
    stage('Deploy Prod'){
		input(message: "Aprovar?",ok: "Aprovado")
		try{
			sh "docker rm -f ivankimura_app_prod"
		} catch (Exception  e) {
			sh "echo $e"
		}
        sh """
            echo "Deploy prod"
			docker run -d -p 81:80 --name=ivankimura_app_prod ivankimura/unyleya:$BUILD_NUMBER
        """
    }
}