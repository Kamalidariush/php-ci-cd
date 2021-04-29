def getdockertag(){
    return "${env.GIT_BRANCH}".replace("/",".") + "."+"${env.BUILD_ID}"
}    
pipeline {
    agent any
    environment {
        DOCKER_REGISTRY = "varunpalekar1/php-test"
        DOCKER_TAG = getdockertag()
    }
    stages {
        stage('Build') {
            steps {
                script {
                    echo "install compose.json"
                    sh 'composer install --prefer-source'
                    sh 'printenv'
                    dir('ansible') {
                        sh 'ansible-galaxy install -r requirements.yml'
                    }
                }
            }
        }
        
        stage('DockerPush') {
            when {
                expression {
                    GIT_BRANCH ==~ /.*master|.*release\/.*|.*develop|.*hotfix\/.*/
                }
            }
            steps {
                script{
                    docker.withRegistry('', 'public-docker-hub') {
                        
                        def customImage = docker.build("${env.DOCKER_REGISTRY}:${env.DOCKER_TAG}")
                        customImage.push()

                        if ( GIT_BRANCH ==~ /.*master|.*hotfix\/.*|.*release\/.*/ )
                            customImage.push('latest')
                    }
                }
            }
        }
        stage('Deploy_Dev') {
            when {
                expression {
                    GIT_BRANCH ==~ /.*develop/
                }
            }
            steps {
                script{
                    echo "Deploy application on developmment environment"
                    dir("ansible") {
                        ansiblePlaybook installation: 'ansible', inventory: 'hosts-dev', playbook: 'playbook.yml', extraVars: [
                            deployment_app_image: "${env.DOCKER_REGISTRY}:${env.DOCKER_TAG}"
                        ]
                    }
                }

                input message: "Do you want to run migration?"

                script{
                    echo "Deploy application on developmment environment"
                    dir("ansible") {
                        ansiblePlaybook installation: 'ansible', inventory: 'hosts-dev', playbook: 'playbook.yml', tags: 'migration'
                    }
                }

                input message: "Do you want to run seeding?"

                script{
                    echo "Deploy application on developmment environment"
                    dir("ansible") {
                        ansiblePlaybook installation: 'ansible', inventory: 'hosts-dev', playbook: 'playbook.yml', tags: 'seeding'
                    }
                }
            }
        }
        
        stage('Undeploy_Dev'){
            when {
                expression {
                    GIT_BRANCH ==~ /.*develop/
                }
            }
            // timeout(time:5, unit:'DAYS') {
            //     input message:'Approve deployment?', submitter: 'it-ops'
            // }
            steps {
                input message: "Do you want to undeploy DEV?"
                script {
                    echo "Undeploy application on developmment environment"
                    dir("ansible") {
                        ansiblePlaybook installation: 'ansible', inventory: 'hosts-dev', playbook: 'playbook.yml', tags: 'undeploy'
                    }
                }
            }
        }
        stage('Deploy_Prod') {
            when {
                expression {
                    GIT_BRANCH ==~ /.*master|.*release\/.*|.*hotfix\/.*/
                }
            }
            steps {
                input message: "Do you want to proceed for production deployment?"
                script{
                    echo "Deploy application on stage environment"
                    dir("ansible") {
                        ansiblePlaybook installation: 'ansible', inventory: 'hosts-prod', playbook: 'playbook.yml', credentialsId: 'ansible-hospice-prod'
                    }
                }

                input message: "Do you want to proceed for production migration?"
                script{
                    echo "Deploy application on stage environment"
                    dir("ansible") {
                        ansiblePlaybook installation: 'ansible', inventory: 'hosts-prod', playbook: 'playbook.yml', tags: 'migration', credentialsId: 'ansible-hospice-prod'
                    }
                }
            }
        }

    }
}
