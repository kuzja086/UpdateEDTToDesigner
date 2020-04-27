
def BIN_CATALOG = ''
def ACC_PROPERTIES = ''
def ACC_BASE = ''
def ACC_USER = ''
def BSL_LS_PROPERTIES = ''
def CURRENT_CATALOG = ''
def TEMP_CATALOG = ''
def PROJECT_NAME_EDT = ''
def PROJECT_KEY
def EDT_VALIDATION_RESULT = ''
def GENERIC_ISSUE_JSON = ''
def SRC = ''
def PROJECT_URL = ''

pipeline {

    parameters {
        string(defaultValue: "${env.PROJECT_NAME}", description: '* Имя проекта в EDT', name: 'PROJECT_NAME')
        string(defaultValue: "${env.git_repo_url}", description: '* URL к гит-репозиторию, который необходимо проверить.', name: 'git_repo_url')
        string(defaultValue: "${env.git_repo_branch}", description: 'Ветка репозитория, которую необходимо проверить. По умолчанию master', name: 'git_repo_branch') 
        string(defaultValue: "${env.jenkinsAgent}", description: 'Нода дженкинса, на которой запускать пайплайн. По умолчанию master', name: 'jenkinsAgent')
        string(defaultValue: "${env.EDT_VERSION}", description: 'Используемая версия EDT. По умолчанию 2020.3', name: 'EDT_VERSION')
    }
    agent {
        label "${(env.jenkinsAgent == null || env.jenkinsAgent == 'null') ? "master" : env.jenkinsAgent}"
    }
    options {
        timeout(time: 8, unit: 'HOURS')
        buildDiscarder(logRotator(numToKeepStr: '10'))
    }
    stages {
        stage("Инициализация переменных") {
            steps {
                timestamps {
                    script {

                        git_repo_branch = git_repo_branch.isEmpty() ? 'master' : git_repo_branch
                        
                        SRC = "./${PROJECT_NAME}/src"

                        CURRENT_CATALOG = pwd()
                        TEMP_CATALOG = "${CURRENT_CATALOG}\\temp"
                        CURRENT_CATALOG = "${CURRENT_CATALOG}\\Repo"
                        XMLPATH = "${TEMP_CATALOG}\\xmlpath"

                        // создаем/очищаем временный каталог
                        dir(TEMP_CATALOG) {
                            deleteDir()
                            dir(XMLPATH){}
                        }
                        PROJECT_NAME_EDT = "${CURRENT_CATALOG}\\${PROJECT_NAME}"

                        EDT_VERSION = EDT_VERSION.isEmpty() ? '2020.3' : EDT_VERSION
                    }
                }
            }
        }
        stage('Проверка изменений проекта') {
            steps {
                timestamps {
                    script {
                        dir('Repo') {
                            checkout([$class: 'GitSCM',
                            branches: [[name: "*/${git_repo_branch}"]],
                            doGenerateSubmoduleConfigurations: false,
                            extensions: [[$class: 'CheckoutOption', timeout: 60], [$class: 'CloneOption', depth: 0, noTags: true, reference: '', shallow: false, timeout: 60]],
                            submoduleCfg: [],
                            userRemoteConfigs: [[url: git_repo_url]]])
                        }
                    }
                }
            }
        }
        stage('Выгрузка проекта из EDT в файлы конфигурации') {
            steps {
                timestamps {
                    script {
                        cmd("""
                        ring edt@${EDT_VERSION} workspace export --workspace-location \"${TEMP_CATALOG}\" --project \"${PROJECT_NAME_EDT}\" --configuration-files \"${TEMP_CATALOG}\
                        """)
                   }
                }
            }
        }
    }
}
def cmd(command) {
    // при запуске Jenkins не в режиме UTF-8 нужно написать chcp 1251 вместо chcp 65001
    if (isUnix()) { sh "${command}" } else { bat "chcp 1251\n${command}" }
}
