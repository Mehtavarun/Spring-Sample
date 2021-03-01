def getPortOnEnv(BRANCH_NAME) {
	if (BRANCH_NAME == 'P-Test') {
                return 6200
            } else {
                return 6000
            }
}
pipeline {
    environment {
	    dockerRegistry = "dtr.nagarro.com:443"
            gitrepo = "https://github.com/Mehtavarun/Spring-Sample.git"
	    userName = "varunmehta02"
	    BRANCH_NAME = "${buildEnv}"
            PORT = getPortOnEnv(BRANCH_NAME);
  	}

    agent any

    tools { 
        maven "Maven3"
    }

    options {
        
        timeout(time: 1, unit: "HOURS")

        skipDefaultCheckout()
    }

    stages {
        stage('Checkout') {
            steps {
                
                checkout([$class: 'GitSCM', branches: [[name: "*/${buildEnv}"]], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[credentialsId: '69231027-d87e-4cbb-a81f-4cbd673533d1', url: 'https://git.nagarro.com/NAGP/varunmehta02.git/']]])

            }
        }
        stage('Build') {
            steps {

                bat "mvn install"

            }
        }
        stage('Unit Test') {
		    steps {
		        
				bat "mvn test"
				
			}
        }
        stage('Sonar Analysis') {
            steps {
                
                withSonarQubeEnv('Test_Sonar'){

                    bat "mvn sonar:sonar -Dsonar.test.exclusions=**/test/**/*.* -Dsonar.exclusions=**/ai/**/*.*,**/jdbc/**/*.*,**/mpt/**/*.*,**/jcr/**/*.*,**/*.c,**/*.java,**/*.h,**/*.cc,**/*.cpp,**/*.cxx,**/*.c++,**/*.hh,**/*.hpp,**/*.hxx,**/*.h++,**/*.ipp,**/*.m"
                }
                
            }
        }
        stage('Push to Artifactory'){
            steps {
                rtMavenDeployer(
                    id: 'deployer',
                    serverId: '123456789@artifactory',
                    releaseRepo: 'CI-Automation-JAVA',
                    snapshotRepo: 'CI-Automation-JAVA'
                    )
                rtMavenRun(
                    pom: 'pom.xml',
                    goals: 'clean install',
                    deployerId: 'deployer'
                    )
                rtPublishBuildInfo(
                    serverId: '123456789@artifactory'
                    )
            }
        }
        stage('Create Docker Image') {
            steps {
                
                    bat "docker build -t ${dockerRegistry}/i_${userName}_${BRANCH_NAME}:${Build_NUMBER} --no-cache -f Dockerfile ."
                
            }
        }
        stage('Push to DTR') {
            steps {

                    bat "docker push ${dockerRegistry}/i_${userName}_${BRANCH_NAME}:${Build_NUMBER}"
                    
            }
        }
        stage('Stop and Remove Container') {
            steps {
                
                bat "docker ps -aq --filter \"name=c_${userName}_${BRANCH_NAME}\" | (findstr . && docker stop c_${userName}_${BRANCH_NAME} && docker rm -fv c_${userName}_${BRANCH_NAME}) || echo No Container Running with name c_${userName}_${BRANCH_NAME}"
                
            }
        }
        stage('Run New Container') {
            steps {
                
                bat "docker run -d --name c_${userName}_${BRANCH_NAME} -p 6200:8080 ${dockerRegistry}/i_${userName}_${BRANCH_NAME}:${Build_NUMBER}"
                
            }
        }
        stage('Helm Deployment'){
			steps {

				bat "helm upgrade --install nagp-helm-chart-${userName} nagp-helm-chart-${userName} --set imageName=dtr.nagarro.com:443/i_${userName}_${BRANCH_NAME}:${BUILD_NUMBER}"
			
			}
		}
    }
}
