@Library('defra-library@v-9') _
node {
  git url: 'https://github.com/DEFRA/ffc-git-secret-scanning.git'
  def dockerImgName = "trufflehog"
  def excludeStrings = []
  stage("Build truffleHog docker image") {
    sh "docker build . --tag $dockerImgName"
  }
  stage("Set exclude strings file") {
    if (params.applyExcludeStrings) {
      def fileContent = readFile "exclude-strings.txt"
      fileContent.split().each {
        excludeStrings.add(it)
      }
    }
    echo "EXCLUDE STRINGS = $excludeStrings"
  }
  stage("Run secret scanner") {
    def secretsFound = false
    try {
      secretsFound = secretScanner.scanFullHistory(
        'github-auth-token',
        dockerImgName,
        params.githubUser,
        params.repoPrefix,
        excludeStrings,
        params.slackChannel)
    } finally {
      if (secretsFound) {
        throw new Exception("Potential secret/s found in scan")
      }
    }
  }
}
