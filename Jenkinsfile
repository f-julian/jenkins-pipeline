#!groovy
def countries = ['be', 'ch', 'nl']

stage('Build') {
    node {
        echo 'checkout'
        // if master or release
        echo 'update pom version of not master or release'
        echo 'mvn deploy'
        bat 'set'
        checkout scm
}
stage('IT-Test') {
    milestone label: 'IT'

    def tasks = [:]
    for (i = 0; i <countries.size(); i++) {
       def country = countries[i]
       tasks.put(country, {
           node {
                   lock(quantity: 1, label: 'mimas_it') {
                     echo 'checkout'
                     checkout scm
                     echo 'got: ' + org.jenkins.plugins.lockableresources.LockableResourcesManager.class.get().getResourcesFromBuild(currentBuild.getRawBuild())[0].getName()
                     echo "bootstrap $country"
                     echo "it test $country"
                   }
           }
           })
    }

        parallel (tasks)
}
stage('UI-Test') {
    input message: 'Deploy?'
    milestone label: 'UI'

    for (i = 0; i <countries.size(); i++) {
       tasks.put(countries[i], {
           lock(quantity: 1, label: 'mimas_feature_dev_env_be') {
               build job: 'deploy', parameters: [string(name: 'ENV_NAME', value: BRANCH_NAME),
                                                 string(name: 'DB_USER', value: 'lockedresource'),
                                                 string(name: 'COUNTRY', value: 'be')]
               node {
                   echo 'checkout'
                   echo 'geb be' // traefik url of env
               }
               // build job undeploy
               }
           })
    }
    parallel (tasks)
  }
