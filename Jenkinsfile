
pipeline {
    agent any
    
    tools {
      maven 'maven'
    }
    
    environment {
        server_name = "test-jar-1"
         git_url = "git@codeup.teambition.com:5fb4ee9a20403ea6dc76b401/mvn_test.git"
        server_port = "43001"
        env="prod"
         dockerfile="/data/srv/docker/${server_name}/Dockerfile"
        namespace="default"
        
        image_name = "registry.cn-beijing.aliyuncs.com/wangluyu/project:${server_name}_${BUILD_NUMBER}"
        images_horbor = "registry.cn-beijing.aliyuncs.com"
        git_auth = "85082035-a847-48e2-b89a-e60b2c908072"
        
    }
    
    parameters {
        gitParameter branch: '', branchFilter: '.*', defaultValue: 'origin/master', 
        description: '选择您要发布的分支，默认分支为master', name: 'faban', 
        quickFilterEnabled: false, selectedValue: 'NONE', sortMode: 'NONE', tagFilter: '*', type: 'PT_BRANCH'
        
        
          choice choices: ['Deploy', 'Rollback'], description: 
                            '''
                            Deploy -- 发版
                            Rollback  -- 回滚
                            ''', 
                            name: 'Status'
        
    }
    
    
    
    
    stages {
        stage('git pull') {
            steps {
                checkout([$class: 'GitSCM', branches: [[name: "${params.faban}" ]], extensions: [], 
                userRemoteConfigs: [[credentialsId: "${git_auth}", 
                url: "${git_url}"]]])
            }
        }
        
        stage('code to package') {

            when {
                environment name: 'Status', value: 'Deploy'
                    }
                    
                    
            steps {
                sh """
                    mvn clean install
                    
                    """
            }
        }
        
        stage('docker build and push') {
            
            when {
                environment name: 'Status', value: 'Deploy'
                    }            
            
            steps {
                    withCredentials([usernamePassword(credentialsId: 'bb193478-a220-4661-ab91-bd7bf2292753', passwordVariable: 'password', usernameVariable: 'username')]) {
                    // some block
                    sh """
                        docker login -u ${username} -p "${password}" ${images_horbor} | true
                        cp ${dockerfile} .
                        sed -i "3{s/test/${env}/}" Dockerfile
                        cat Dockerfile
                        docker build -t "${image_name}" .
                        docker push ${image_name} && docker rmi ${image_name} 
                    """
                    }
                    
                    
            }
        }
        
        stage('deployment') {
            
            when {
                environment name: 'Status', value: 'Deploy'
                    }
                    
            steps {
                sh """
                    sed -i 's#myapp#${server_name}#g' deploy.yaml
                    sed -i 's#IMAGE_URL#${image_name}#g' deploy.yaml
                    cat deploy.yaml
                """
                configFileProvider([configFile(fileId: '11043090-c298-4404-a47c-6f7fcaa7181a', targetLocation: 'kube.config')]) {
                // some block
                
                    sh """
                    common_args=" --kubeconfig kube.config -n ${namespace}"
                    # kubectl \${common_args} apply -f deploy.yaml
                    kubectl \${common_args} set image deployment/"${server_name}" "${server_name}"="${image_name}"
                    """
                
                }
                
                
            }
        }
        
        stage('Rollback') {
            
            when {
                environment name: 'Status', value: 'Rollback'
                    }
                    
            steps {
                configFileProvider([configFile(fileId: '11043090-c298-4404-a47c-6f7fcaa7181a', targetLocation: 'kube.config')]) {
                // some block
                
                    sh """
                        common_args=" --kubeconfig kube.config -n ${namespace}"
                        kubectl \${common_args} rollout undo deployment/${server_name}
                    """
                
                }
                
                
            }
        }
        
        
    }
}
