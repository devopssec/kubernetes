node('jnlp') {
    stage('Prepare') {
        echo "1.Prepare Stage"
        checkout scm
        script {
            build_tag = sh(returnStdout: true, script: 'git rev-parse --short HEAD').trim()
            if (env.BRANCH_NAME != 'master') {
                build_tag = "${env.BRANCH_NAME}-${build_tag}"
            }
        }
    }
    stage('Test') {
        echo "2.Test Stage"
    }
    stage('Build') {
        echo "3.Build Docker Image Stage"
        sh 'echo "192.168.56.21  harbor.devopssec.cn" >>/etc/hosts'
        sh "docker build -t harbor.devopssec.cn/demo/jenkins-demo:${build_tag} jenkins/jenkins-demo/."
    }
    stage('Push') {
        echo "4.Push Docker Image Stage"
        withCredentials([usernamePassword(credentialsId: 'harbor', passwordVariable: 'harborPassword', usernameVariable: 'harborUser')]) {
            sh "docker login harbor.devopssec.cn -u ${harborUser} -p ${harborPassword}"
            sh "docker push harbor.devopssec.cn/demo/jenkins-demo:${build_tag}"
        }
    }
    stage('Deploy') {
        echo "5. Deploy Stage"
        if (env.BRANCH_NAME == 'master') {
            input "确认要部署线上环境吗？"
        }
        sh "sed -i 's/<BUILD_TAG>/${build_tag}/' jenkins/jenkins-demo/deployment.yaml"
        sh "sed -i 's/<BRANCH_NAME>/${env.BRANCH_NAME}/' jenkins/jenkins-demo/deployment.yaml"
        sh "kubectl apply -f jenkins/jenkins-demo/deployment.yaml --record"
    }
}
