pipeline {
    agent none
    
    environment {
        P_VERSION='3.0.3'
    }

    stages{
        stage('CI') {
            agent{ label 'win-agent1' }
            stages{
                stage('clean-workspace'){
                    steps {
                        cleanWs();
                    }
                }
                stage('checkout'){
                  steps {
                      checkout scmGit(branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[credentialsId: 'jojo-987', url: 'https://github.com/jojo-987/basic-vite.git']])
                  }
                }
                stage('install_deps'){
                    steps{
                         bat 'npm i'
                    }
                }
                stage('build'){
                    steps{
                        bat 'npm run build'
                    }
                }
                stage('archive'){
                    steps{
                    zip defaultExcludes: false, dir: './dist', exclude: '', glob: '', zipFile: "./basic-vite-${P_VERSION}-${BUILD_TIMESTAMP}.zip"
                    }
                }
                stage('deploy-nexus'){
                    steps {
                    nexusArtifactUploader artifacts: [[
                    artifactId: 'basic-vite', 
                    classifier: '', 
                    file: "./basic-vite-${P_VERSION}-${BUILD_TIMESTAMP}.zip", 
                    type: 'zip'
                    ]], 
                    credentialsId: 'central-nexus', 
                    groupId: 'krish.frontend', 
                    nexusUrl: '192.168.0.49:8081', 
                    nexusVersion: 'nexus3', 
                    protocol: 'http', 
                    repository: 'Interns_Nexus_2026', 
                    version: "${P_VERSION}-${BUILD_TIMESTAMP}"
                    }
                }
                
            }
            
        }
        stage('CD') {
            agent{ label 'pm' }
            stages{
                stage('fetch-playbook'){
                    steps{
                        checkout scmGit(branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[credentialsId: 'jojo-987', url: 'https://github.com/jojo-987/ansible-playbooks.git']])
                    }
                }
                stage('running-playbook'){
                    steps {
                    ansiblePlaybook credentialsId: '192.168.0.202', 
                    disableHostKeyChecking: true, 
                    installation: 'ansible', 
                    playbook: './final-redeploy-frontend-playbook.yml', 
                    vaultTmpPath: '',
                    extraVars: [
                        nexus_url: 'http://192.168.0.49:8081/service/rest/v1/search/assets/download?sort=version&direction=desc&repository=Interns_Nexus_2026&maven.extension=zip&group=krish.frontend&name=basic-vite',
                        project_name: 'basic-vite',
                        site_name: 'kp-basic-vite2',
                        site_url: 'http://192.168.0.116:5173',
                        host_port: '5173'
                        ]
                    }
                }
            }
    }
    }
}
