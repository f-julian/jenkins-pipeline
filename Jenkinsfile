#!groovy
def countries = ['be', 'ch', 'nl']
def autoDeploy = isMaster() || isReleaseBranch()
def autoUndeploy = !isMaster()
def updateVersion = !isMaster() && !isReleaseBranch()

def buildTag

echo "isMaster: ${isMaster()}"
echo "isReleaseBranch: ${isReleaseBranch()}"

stage('Build') {
    node {
        checkout scm

        buildTag = "build/${BRANCH_NAME}/${BUILD_NUMBER}"
        bat "git tag $buildTag"
        bat "git pubat origin $buildTag"

        // if master or release
        if (updateVersion) {
            echo 'update pom version'
        }

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
                    bat "git checkout $buildTag"

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

            withTestEnv(country, {envName, dbUser, envUrl ->
                build job: 'deploy', parameters: [string(name: 'ENV_NAME', value: envName),
                                                  string(name: 'DB_USER', value: dbUser),
                                                  string(name: 'COUNTRY', value: country)]
                node {
                    echo 'checkout'
                    echo "geb on $envUrl"
                }

                if (!autoUndeploy) {
                    input message: 'Undeploy?'
                }
                // build job undeploy
            })


        })
    }
    parallel(tasks)
}

def withTestEnv(country, task) {

    if (isMaster()) {
        def dbUser = "ci_$country"
        def envName = envNameForBranch(BRANCH_NAME)
        def envUrl = "${country}.cosmolb.mgm-edv.de/mimas/ci"
        task.call(envName, dbUser, envUrl)
    } else {
        lock(quantity: 1, label: 'mimas_dev_env') {
            def dbUser = lockedResource()
            def envName = envNameForBranch(BRANCH_NAME)
            def envUrl = "${country}.cosmolb.mgm-edv.de/mimas/$envName"
            task.call(envName, dbUser, envUrl)
        }
    }
}

def lockedResource() {
    org.jenkins.plugins.lockableresources.LockableResourcesManager.class.get().getResourcesFromBuild(currentBuild.getRawBuild())[0].getName()
}

def envNameForBranch(branch) {

    if (branch == 'master') {
        return 'ci'
    }

    def group = (branch =~ /(.+)\/(.+)/)

    def type = group[0][1]
    def qualifier = group[0][2]

    if (type == 'release') {
        return "rel-$qualifier"
    }

    return qualifier
}

def isMaster() {
    BRANCH_NAME == 'master'
}

def isReleaseBranch() {
    def group = (BRANCH_NAME ==~ /release\/.+/)
    return group
}