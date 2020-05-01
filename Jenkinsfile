@Library("shared-libraries")
import io.libs.ProjectHelpers
import io.libs.Utils

def CURRENT_CATALOG = ''
def TEMP_CATALOG = ''
def SRC = ''
def shortPlatformName = ''
def projectHelpers = new ProjectHelpers()
def utils = new Utils()

pipeline {

    parameters {
        string(defaultValue: "${env.PROJECT_NAME}", description: '* Имя проекта в EDT', name: 'PROJECT_NAME')
        string(defaultValue: "${env.git_repo_url}", description: '* URL к гит-репозиторию, который необходимо проверить.', name: 'git_repo_url')
        string(defaultValue: "${env.git_repo_branch}", description: 'Ветка репозитория, которую необходимо проверить. По умолчанию master', name: 'git_repo_branch') 
        string(defaultValue: "${env.jenkinsAgent}", description: 'Нода дженкинса, на которой запускать пайплайн. По умолчанию master', name: 'jenkinsAgent')
        string(defaultValue: "${env.EDT_VERSION}", description: 'Используемая версия EDT. По умолчанию 2020.3', name: 'EDT_VERSION')
        string(defaultValue: "${env.PLATFORM1C}", description: 'Используемая платформа. По умолчанию 8.3.14.1779', name: 'PLATFORM1C')
        string(defaultValue: "${env.SERVER1C}", description: 'Адрес сервера 1С. По умолчанию localhost', name: 'SERVER1C')
        string(defaultValue: "${env.PORT1C}", description: 'Порт агента кластера 1с. По умолчанию 1541', name: 'PORT1C')
        string(defaultValue: "${env.BASE1C}", description: 'Имя базы для загрузки из файлов', name: 'BASE1C')
        string(defaultValue: "${env.USER1C}", description: 'Имя пользователя базы 1с', name: 'USER1C')
        string(defaultValue: "${env.PWD1C}", description: 'Пароль пользователя', name: 'PWD1C')
        string(defaultValue: "${env.CFPATH}", description: 'Каталог для сохранения файла .cf. Например: D:\\1c. По Умолчанию в Воркспейсе ноды', name: 'CFPATH')
        string(defaultValue: "${env.git_credentials_Id}", description: 'ID Credentials для получения изменений из гит-репозитория', name: 'git_credentials_Id')
        string(defaultValue: "${env.Repository}", description: 'Путь к хранилищу конфигурации', name: 'Repository')
        string(defaultValue: "${env.UserRepository}", description: 'Пользователь хранилища', name: 'UserRepository')
        string(defaultValue: "${env.PWDRepository}", description: 'Пароль пользователя в хранилище', name: 'PWDRepository')
        booleanParam(defaultValue: env.Check == null ? true : env.Check, description: 'Выполнять проверку кода. По умолчанию Истина', name: 'Check')
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
                        XMLPATH = "${CURRENT_CATALOG}\\xmlpath"
                        CURRENT_CATALOG = "${CURRENT_CATALOG}\\Repo"
                        NAMECF = "\\${PROJECT_NAME}.cf"
                        CATALOGCF = pwd()
                        
                        EDT_VALIDATION_RESULT = "${TEMP_CATALOG}\\edt-result.csv"
                        BSL_LS_PROPERTIES = "./Sonar/bsl-language-server.conf"
                        BSL_LS = "./Sonar/bsl-language-server.jar"
                        GENERIC_ISSUE_JSON ="${TEMP_CATALOG}/bsl-generic-json.json,${TEMP_CATALOG}/edt.json"
                        STEBI_SETTINGS = "./Sonar/settings.json"

                        // создаем/очищаем временный каталог
                        dir(TEMP_CATALOG) {
                            deleteDir()
                        }
                        dir(XMLPATH){
                            deleteDir()
                        }

                        PROJECT_NAME_EDT = "${CURRENT_CATALOG}\\${PROJECT_NAME}"

                        EDT_VERSION = EDT_VERSION.isEmpty() ? "2020.3" : EDT_VERSION

                        SERVER1C = SERVER1C.isEmpty() ? 'localhost' : SERVER1C
                        PORT1C = PORT1C.isEmpty() ? '1541' : PORT1C
                        PLATFORM1C = PLATFORM1C.isEmpty() ? '8.3.14.1779' : PLATFORM1C
                        baseconnstring = projectHelpers.getConnString(SERVER1C, BASE1C, PORT1C)

                        CFPATH = CFPATH.isEmpty() ? "${CATALOGCF}${NAMECF}" : "${CFPATH}${NAMECF}"

                        shortPlatformName = utils.shortPlatformName(PLATFORM1C)
                        IB = "File=${TEMP_CATALOG}"
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
                            userRemoteConfigs: [[credentialsId: git_credentials_Id, url: git_repo_url]]])
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
                        @set RING_OPTS=-Dfile.encoding=UTF-8 -Dosgi.nl=ru
                        ring edt@${EDT_VERSION} workspace export --workspace-location \"${TEMP_CATALOG}\" --project \"${PROJECT_NAME_EDT}\" --configuration-files \"${XMLPATH}\
                        """)
                   }
                }
            }
        }
         stage('Проверка кода') {
            steps {
                timestamps {
                    script {
                        //Надо сделать цикл по списку баз
                        if (${Check}==true) {
                            cmd("""
                            @set RING_OPTS=-Dfile.encoding=UTF-8 -Dosgi.nl=ru
                            ring edt@${EDT_VERSION} workspace validate --workspace-location \"${TEMP_CATALOG}\" --file \"${EDT_VALIDATION_RESULT}\" --project-list \"${PROJECT_NAME_EDT}\"
                            set SRC=\"./Repo/${SRC}\"
                            stebi convert -e \"${EDT_VALIDATION_RESULT}\" \"${TEMP_CATALOG}/edt.json\"
                            @echo EDT Check complited. 
                            """) 
                            
                            //Проверка BSL
                            cmd("""
                            java -Xmx16g -jar ${BSL_LS} -a -s \"./Repo/${SRC}\" -r generic -c \"${BSL_LS_PROPERTIES}\" -o \"${TEMP_CATALOG}\"
                            @echo BSL Check complited.
                            """)

                            //Трансформация результатов
                            cmd("""
                            set GENERIC_ISSUE_SETTINGS_JSON=\"${STEBI_SETTINGS}\"
                            set GENERIC_ISSUE_JSON=${GENERIC_ISSUE_JSON}
                            set SRC=\"./Repo/${SRC}\"
                            stebi transform -r=0
                            """)
                        }
                   }
                }
            }
        }
        stage('Загрузка конфигурации из файлов') {
            steps {
                timestamps {
                    script {
                        cmd("""
                        cd /D C:\\Program Files (x86)\\1cv8\\${PLATFORM1C}\\bin\\
                        1cv8.exe CREATEINFOBASE ${IB}
                        1cv8.exe DESIGNER /WA- /DISABLESTARTUPDIALOGS /IBConnectionString ${IB} /LoadConfigFromFiles ${XMLPATH} /UpdateDBCfg
                        """)
                   }
                }
            }
        }
         stage('Сохранение файла .cf') {
            steps {
                timestamps {
                    script {
                        cmd("""
                        cd /D C:\\Program Files (x86)\\1cv8\\${PLATFORM1C}\\bin\\
                        1cv8.exe DESIGNER /WA- /DISABLESTARTUPDIALOGS /IBConnectionString ${IB} /CreateDistributionFiles -cffile ${CFPATH}
                        """)
                   }
                }
            }
        }
        stage('Сравнение/Объединение с базой в хранилище') {
            steps {
                timestamps {
                    script {
                        //Надо сделать цикл по списку баз
                        if (REPOSITORY != null && !REPOSITORY.isEmpty() && USERREPOSITORY != null && !USERREPOSITORY.isEmpty() && PWDREPOSITORY != null && PWDREPOSITORY.isEmpty()) {
                            cmd("""
                            cd /D C:\\Program Files (x86)\\1cv8\\${PLATFORM1C}\\bin\\
                            1cv8.exe DESIGNER /WA- /DISABLESTARTUPDIALOGS ${baseconnstring} ${USER1C} ${PWD1C} /ConfigurationRepositoryF ${REPOSITORY} /ConfigurationRepositoryN ${USERREPOSITORY} /ConfigurationRepositoryP ${PWDREPOSITORY} /ConfigurationRepositoryUnLock -force
                            """)
                        }

                        cmd("""
                        cd /D C:\\Program Files (x86)\\1cv8\\${PLATFORM1C}\\bin\\
                        1cv8.exe DESIGNER /WA- /DISABLESTARTUPDIALOGS ${baseconnstring} ${USER1C} ${PWD1C} /MergeCfg ${CFPATH} -force /UpdateCfg
                        """)
                   }
                }
            }
        }
    }
}
def cmd(command) {
    // при запуске Jenkins не в режиме UTF-8 нужно написать chcp 1251 вместо chcp 65001
    if (isUnix()) { sh "${command}" } else { bat "chcp 65001\n${command}" }
}
