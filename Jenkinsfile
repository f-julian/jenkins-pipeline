#!groovy
def countries = ['be', 'ch', 'nl']
def autoDeploy = isMaster() || isReleaseBranch()
def autoUndeploy = !isMaster()


stage('Build') {
    node {
        checkout scm

        // if master or release
        echo 'update pom version of not master or release'
        echo 'mvn deploy'
        bat 'set'
    }
}

stage('IT-Test') {
    milestone label: 'IT'

    def tasks = [:]
    for (i = 0; i < countries.size(); i++) {
        def country = countries[i]
        tasks.put(country, {
            node {
                lock(quantity: 1, label: 'mimas_it') {
                    checkout scm
                    echo 'got: ' + lockedResource()
                    echo "bootstrap $country"
                    echo "it test $country"
                }
            }
        })
    }

    parallel(tasks)
}

stage('UI-Test') {
    if (!autoDeploy) {
        input message: 'Deploy?'
    }

    milestone label: 'UI'

    def tasks = [:]
    for (i = 0; i < countries.size(); i++) {
        def country = countries[i]
        tasks.put(countries[i], {
            lock(quantity: 1, label: 'mimas_feature_dev_env') {
                def lockedResource = lockedResource()
                def envName = envNameForBranch(BRANCH_NAME)

                build job: 'deploy', parameters: [string(name: 'ENV_NAME', value: envName),
                                                  string(name: 'DB_USER', value: lockedResource),
                                                  string(name: 'COUNTRY', value: country)]
                node {
                    echo 'checkout'
                    echo 'geb be' // traefik url of env
                }

                if (!autoUndeploy) {
                    input message: 'Undeploy?'
                }
                // build job undeploy
            }
        })
    }
    parallel(tasks)
}


def lockedResource() {
    org.jenkins.plugins.lockableresources.LockableResourcesManager.class.get().getResourcesFromBuild(currentBuild.getRawBuild())[0].getName()
}

def envNameForBranch(branch) {

    def group = (branch =~ /(.+)\/(.+)/)

    def type = group[0][1]
    def qualifier = group[0][2]

    if (type == 'master') {
        return 'ci'
    }
    if (type == 'release') {
        return "rel-$qualifier"
    }

    return qualifier
}

def isMaster() {
    BRANCH_NAME == 'master'
}

def isReleaseBranch() {
    BRANCH_NAME =~ /release\/.+/
}