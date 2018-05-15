node {
  sh("whoami")
  sh("docker version")
  sh("kubectl version")
  sh("echo BRANCH_NAME=${env.BRANCH_NAME}")
  sh("echo BUILD_NUMBER=${env.BUILD_NUMBER}")
  
  def project = 'Kubernetes - Jenkins for CI/CD'
  def appName = 'cotd'
  def imageTag = "stefanopicozzi/cotd:${env.BRANCH_NAME}.${env.BUILD_NUMBER}"

  checkout scm

  stage 'Build image'
    sh("cat ./etc/config/cotd.properties")
    sh("docker build -t ${imageTag} .")
  
  stage 'Deploy Application'
    switch (env.BRANCH_NAME) {  

      // Roll out to canary
      case "canary":
        // Change deployed image in canary to the one we just built
        sh("sed -i.bak 's#stefanopicozzi/cotd#${imageTag}#' ./etc/kubernetes/jenkins/canary/cotd.yaml")
        sh("kubectl --namespace=production apply -f etc/kubernetes/jenkins/canary/cotd.yaml")
        sh("kubectl scale deployment cotd-canary --namespace=production --replicas=1")
        sh("kubectl describe pod cotd-canary --namespace=production")
        break

      // Roll out to production
      case "master":
        // Change deployed image in canary to the one we just built
        sh("sed -i.bak 's#stefanopicozzi/cotd#${imageTag}#' ./etc/kubernetes/jenkins/production/cotd.yaml")
        sh("kubectl --namespace=production apply -f etc/kubernetes/jenkins/production/cotd.yaml")
        sh("kubectl --namespace=production apply -f etc/kubernetes/jenkins/services/production.yaml")
        sh("kubectl scale deployment cotd-production --namespace=production --replicas=4")
        sh("kubectl describe pod cotd-production --namespace=production")
        break

      // Roll out a dev environment
      default:
        // Create namespace if it doesn't exist
        sh("kubectl get ns ${env.BRANCH_NAME} || kubectl create ns ${env.BRANCH_NAME}")
        sh("sed -i.bak 's#stefanopicozzi/cotd#${imageTag}#' ./etc/kubernetes/jenkins/development/cotd.yaml")
        sh("kubectl --namespace=${env.BRANCH_NAME} apply -f etc/kubernetes/jenkins/development/cotd.yaml")
        sh("kubectl --namespace=${env.BRANCH_NAME} apply -f etc/kubernetes/jenkins/services/development.yaml")
        sh("kubectl scale deployment cotd-development --namespace=${env.BRANCH_NAME} --replicas=1")
        sh("kubectl describe pod cotd-development --namespace=${env.BRANCH_NAME}")
   }
}
