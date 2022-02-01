pipeline {
    agent any
        environment {
         registry = 'lokits/spring-boot'
         registryCredential = 'Docker_Hub_Pwd'
         dockerImage = ''

    }
    parameters {
        string(name: 'git_repo_source', defaultValue: 'https://github.com/lokits/argocd_helm.git', description: 'Git repository from where we are going to checkout the code (dev  branch) and build.')
        string(name: 'git_repo_branch', defaultValue: 'main', description: 'The branch to be checked out')
        string(name: 'helm_repo_source', defaultValue: 'https://github.com/lokits/argocd_helm.git', description: 'Git repository from where we are going to checkout the code (dev  branch) and build.')
        string(name: 'helm_repo_branch', defaultValue: 'main', description: 'The branch to be checked out')
        string(name: 'helm_repo_source_push', defaultValue: 'github.com/lokits/argocd_helm.git', description: 'Git repository from where we are going to checkout the code (dev  branch) and build.')
        credentials(name: 'argocd', defaultValue: 'argocd', description: 'helm repo credentials', credentialType: "Secret text", required: true)
    }
tools{
maven 'maven-3.6.3'
}
    options {
        disableConcurrentBuilds()
        buildDiscarder(logRotator(daysToKeepStr: '7', artifactDaysToKeepStr: '7'))
        timeout(time: 1, unit: 'HOURS')
    }
   stages {
        stage("Checkout git repo") {
            steps {
                script{
                    // cleanWs()
                // env.code_author_name = sh(returnStdout: true, script: "git --no-pager show -s --format='%an' ${commitId}").trim()
                // env.code_author_email = sh(returnStdout: true, script: "git --no-pager show -s --format='%ae' ${commitId}").trim()
                // sh 'echo building image'
                // echo "Removing git references"
                // sh "rm -rf .git"
                git branch: 'main', url: 'https://github.com/lokits/argocd_helm.git'
                     }
                }
            
        } 
        stage('Build') {
            steps{
                sh "mvn clean package"
            }
        }

        stage('Building image') {
            steps{
                script {
                    dockerImage = docker.build registry + ":$BUILD_NUMBER"
                }
            }
        }
        stage('Push Image') {
            steps{
                script {
                     docker.withRegistry( '', registryCredential ) {
                     dockerImage.push()
                    }
                }
            }
        }
        stage('Checkout helm repo') {
            steps {
             git branch: "${params.helm_repo_branch}",
                credentialsId: 'argocd',
                url: "${params.helm_repo_source}"
                withCredentials([usernamePassword(credentialsId: params.argocd, usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]){

                    sh """  sed -i "s/tag: .*\$/tag: DOCKER_IMAGE_TAG/" springboot/values.yaml
                                cat -n springboot/values.yaml | grep -I "tag:" | awk '{print \$1,\$2,\$3}'
                                sed -i "s/DOCKER_IMAGE_TAG.*\$/$BUILD_NUMBER/" springboot/values.yaml
                                cat -n springboot/values.yaml | grep -I "tag:" | awk '{print \$1,\$2,\$3}'
                                apt-get  install -y curl jq python3 python3-pip &&  pip3 install yamllint
                                yamllint -d '{extends: relaxed, rules: {line-length: {max: 250}, trailing-spaces: disable }}' springboot/values.yaml
                        """
                    sh """  git add springboot/values.yaml
                                git config --global user.email "$env.code_author_email"
                                git config --global user.name "$env.code_author_name"
                                git commit -am "Docker Version Change to ${env.DOCKER_IMAGE_TAG} Build ${env.BUILD_NUMBER}"
                                git push https://${USERNAME}:${PASSWORD.replace('@', '%40')}@${params.helm_repo_source_push}
                        """ // Checkout and Commit the version
                }
            }  

        }
        stage('Remove Unused docker image') {
            steps{
             sh "docker rmi $registry:$BUILD_NUMBER"
            }
        }
    }
    
}

