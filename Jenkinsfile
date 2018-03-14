try {
  timeout(time: 20, unit: 'MINUTES') {
    def appName="nodejs-mongodb-example"
    def project=""
    def tag="blue"
    def altTag="green"
    def verbose="false"

    node {
      project = env.PROJECT_NAME
      stage("Initialize") {
        sh "oc get route ${appName} -n ${project} -o jsonpath='{ .spec.to.name }' --loglevel=4 > activeservice"
        activeService = readFile('activeservice').trim()
        if (activeService == "${appName}-blue") {
          tag = "green"
          altTag = "blue"
        }
        sh "oc get route ${tag}-${appName} -n ${project} -o jsonpath='{ .spec.host }' --loglevel=4 > routehost"
        routeHost = readFile('routehost').trim()
      }

      stage("Build") {
        echo "building tag ${tag}"
        openshiftBuild buildConfig: appName, showBuildLogs: "true", verbose: verbose
      }

      stage("Deploy Test") {
        openshiftTag srcStream: appName, srcTag: 'latest', destinationStream: appName, destinationTag: tag, verbose: verbose
        openshiftVerifyDeployment deploymentConfig: "${appName}-${tag}", verbose: verbose
      }

      stage("Test") {
        input message: "Test deployment: http://${routeHost}. Approve?", id: "approval"
      }

      stage("Go Live") {
        sh "oc set -n ${project} route-backends ${appName} ${appName}-${tag}=100 ${appName}-${altTag}=0 --loglevel=4"
      }
    }
  }
  } catch (err) {
  echo "in catch block"
  echo "Caught: ${err}"
  currentBuild.result = 'FAILURE'
  throw err
  }
