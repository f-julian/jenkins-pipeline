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
        parallel (
                be: {
                    node {
                            lock(quantity: 1, label: 'mimas_it') {
                              echo 'checkout'
                              checkout scm
                              echo 'got: ' + org.jenkins.plugins.lockableresources.LockableResourcesManager.class.get().getResourcesFromBuild(currentBuild.getRawBuild())[0].getName()
                              echo 'bootstrap be'
                              echo 'it test be'
                            }
                    }
                },
                ch: {
                    node {
                            lock(quantity: 1, label: 'mimas_it') {
                              echo 'checkout'
                              checkout scm
                              echo 'got: ' + org.jenkins.plugins.lockableresources.LockableResourcesManager.class.get().getResourcesFromBuild(currentBuild.getRawBuild())[0].getName()
                              echo 'bootstrap ch'
                              echo 'it test ch'
                            }
                    }
                },
                failFast: false)
    }
}
stage('UI-Test') {
    input message: 'Deploy?'
    milestone label: 'UI'
    parallel (
        be: {
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

        },
        ch: {
            lock(quantity: 1, resource: 'mimas_feature_dev_env_ch') {
                build job: 'deploy', parameters: [string(name: 'ENV_NAME', value: BRANCH_NAME),
                                                  string(name: 'DB_USER', value: 'lockedresource'),
                                                  string(name: 'COUNTRY', value: 'ch')]
                node {
                    echo 'checkout'
                    checkout scm
                    echo 'geb ch'
                }
            }

        },
        failFast: false)
  }
