# Отчёт



## Необходимо собрать два docker контейнера
Собирать контейнеры не возникло необходимости, функционала официальных образов вполне хватило.

## Первый контейнер должен содержать Gitlab CI/CD с включенным по умолчанию Gitlab Container Registry. Второй контейнер должен содержать gitlab runner.
Используем Docker Compose для настройки и запуска контейнеров (+ запускаем контейнер с SonarQube):

```
$ cat docker-compose.yml
version: '3.6'  # Указывает версию схемы файла docker-compose.yml. В данном случае, это версия 3.6.
services:       # Начало раздела, где определены службы (контейнеры) и их настройки.
  gitlab-1:     # Имя первой службы (контейнера), в данном случае, это контейнер GitLab (экземпляр со включенным по умолчанию Gitlab Container Registry).
    image: 'gitlab/gitlab-ce:16.2.8-ce.0'   # Определяет образ Docker, который будет использоваться для создания контейнера GitLab (с тегом 16.2.8-ce.0).
    restart: always                         # Задает поведение контейнера при его перезапуске. В данном случае, контейнер всегда будет перезапускаться.
    hostname: 'gitlab1.xxx.trash'           # Устанавливает имя хоста (hostname) контейнера.
    environment:                            # Определяет переменные окружения, необходимые для контейнера GitLab.
      GITLAB_OMNIBUS_CONFIG: |              # Здесь заданы дополнительные конфигурационные параметры для GitLab.
        external_url 'http://gitlab1.xxx.trash:8929'            # Задает внешний URL для GitLab.
        registry_external_url "http://registry1.xxx.trash:5050" # Задает внешний URL для Docker Registry и включает Docker Registry в GitLab
    ports:          # Определяет проброс портов между хостовой системой и контейнером.
      - '8929:8929' # Пробрасывает порт 8929 хостовой системы на порт 8929 контейнера.
      - '5050:5050' # Пробрасывает порт 5050 хостовой системы на порт 5050 контейнера.
    volumes:        # Определяет монтирование томов (директорий) в контейнер.
      - './gitlab1/config:/etc/gitlab'    # Монтирует локальную директорию ./gitlab1/config в /etc/gitlab внутри контейнера.
      - './gitlab1/logs:/var/log/gitlab'  # Монтирует локальную директорию ./gitlab1/logs в /var/log/gitlab внутри контейнера.
      - './gitlab1/data:/var/opt/gitlab'  # Монтирует локальную директорию ./gitlab1/data в /var/opt/gitlab внутри контейнера.
    shm_size: '256m'                      # Задает размер разделяемой памяти (SHM) для контейнера.

  gitlab-2:     # Имя второй службы (контейнера), в данном случае, это контейнер GitLab (экземпляр с настройками по умолчанию).
    image: 'gitlab/gitlab-ce:16.2.8-ce.0' # Определяет образ Docker, который будет использоваться для создания контейнера GitLab (с тегом 16.2.8-ce.0).
    restart: always                       # Задает поведение контейнера при его перезапуске. В данном случае, контейнер всегда будет перезапускаться.
    hostname: 'gitlab2.xxx.trash'         # Устанавливает имя хоста (hostname) контейнера.
    environment:                          # Определяет переменные окружения, необходимые для контейнера GitLab.
      GITLAB_OMNIBUS_CONFIG: |            # Здесь заданы дополнительные конфигурационные параметры для GitLab.
        external_url 'http://gitlab2.xxx.trash:8939'  # Задает внешний URL для GitLab.
    ports:          # Определяет проброс портов между хостовой системой и контейнером.
      - '8939:8939' # Пробрасывает порт 8939 хостовой системы на порт 8939 контейнера.
    volumes:        # Определяет монтирование томов (директорий) в контейнер.
      - './gitlab2/config:/etc/gitlab'    # Монтирует локальную директорию ./gitlab2/config в /etc/gitlab внутри контейнера.
      - './gitlab2/logs:/var/log/gitlab'  # Монтирует локальную директорию ./gitlab2/logs в /var/log/gitlab внутри контейнера.
      - './gitlab2/data:/var/opt/gitlab'  # Монтирует локальную директорию ./gitlab2/data в /var/opt/gitlab внутри контейнера.
    shm_size: '256m'                      # Задает размер разделяемой памяти (SHM) для контейнера.

  gitlab-runner:  # Имя службы (контейнера), в данном случае, это контейнер с GitLab Runner.
    image: "gitlab/gitlab-runner:ubuntu-v16.4.0"    # Определяет образ, который будет использоваться для создания контейнера GitLab Runner. В данном случае, используется образ с именем gitlab/gitlab-runner:ubuntu-v16.4.0.
    restart: always                                 # Задает поведение контейнера при его перезапуске. В данном случае, контейнер всегда будет перезапускаться.
    volumes:                                        # Определяет монтирование томов (директорий) в контейнер.
      - './gitlab-runner/config:/etc/gitlab-runner' # Монтирует локальную директорию ./gitlab-runner/config в /etc/gitlab-runner внутри контейнера. Это позволяет передавать конфигурационные файлы и настройки GitLab Runner из локальной директории в контейнер.
      - '/var/run/docker.sock:/var/run/docker.sock' # Монтирует сокет Docker /var/run/docker.sock на сокет /var/run/docker.sock внутри контейнера. Это позволяет GitLab Runner взаимодействовать с Docker на хостовой системе, что необходимо для выполнения CI/CD задач.

  sonarqube:    # Имя службы (контейнера), в данном случае, это контейнер с SonarQube
    image: "sonarqube:9.9.2-community"  # Определяет образ, который будет использоваться для создания контейнера SonarQube. В данном случае, используется образ с Community Edition.
    restart: always                     # Задает поведение контейнера при его перезапуске. В данном случае, контейнер всегда будет перезапускаться.
    hostname: 'sonarqube.xxx.trash'     # Устанавливает имя хоста (hostname) контейнера.
    ports:                              # Определяет проброс портов между хостовой системой и контейнером.
      - '9000:9000'                     # Пробрасывает порт 9000 хостовой системы на порт 9000 контейнера.
    volumes:                            # Определяет монтирование томов (директорий) в контейнер.
      - './sonarqube/sonarqube_data:/opt/sonarqube/data'              # Монтирует локальную директорию ./sonarqube/sonarqube_data в /opt/sonarqube/data внутри контейнера.
      - './sonarqube/sonarqube_extensions:/opt/sonarqube/extensions'  # Монтирует локальную директорию ./sonarqube/sonarqube_extensions в /opt/sonarqube/extensions внутри контейнера.
      - './sonarqube/sonarqube_logs:/opt/sonarqube/logs'              # Монтирует локальную директорию ./sonarqube/sonarqube_logs в /opt/sonarqube/logs внутри контейнера.
```
> [!Замечание]
> в реальной жизни не использую такое избыточное количество комментариев.

## Конфигурация gitlab по включению registry
По умолчанию подсистема "GitLab Container Registry" (базирующаяся на "Docker Registry") отключена. Она активируется, полностью автоматически настраиваясь оркестратором "Omnibus", при обнаружении в конфигурационном файле параметра `registry_external_url`, добавляем в `docker-compose.yml`:
```
registry_external_url "http://registry1.xxx.trash:5050" # Задает внешний URL для Docker Registry и включает Docker Registry в GitLab
```

## В gitlab необходимо создать pipeline по сборке любого контейнера. Gitlab runner после сборки этого контейнера должен положить image в gitlab registry.
Собираем простой hello world контейнер:
```
$ cat Dockerfile
# Используем базовый образ с Python
FROM python:3.8

# Установим рабочую директорию внутри контейнера (+ создание каталога /app)
WORKDIR /app

# Копируем наше hello world Python-приложение в контейнер
COPY ./src/app.py /app

# Команда и параметр для запуска приложения
CMD ["python", "app.py"]
```

## Gitlab runner после сборки этого контейнера должен положить image в gitlab registry.
Пайплайн сборки контейнера (запскается только при создании тега):
```
$ cat .gitlab-ci.yml
stages:     #  Определяет этапы (stages) выполнения пайплайна CI/CD.
  - build   # В данном случае, есть только один этап - build.

build:          # Определяет задачу (job) с именем build.
  stage: build  # Указывает, что задача build принадлежит этапу build.
  image:        # Определяет образ, который будет использоваться для выполнения этой задачи.
    name: gcr.io/kaniko-project/executor:v1.14.0-debug  # Указывает имя Docker-образа. В данном случае, используется образ gcr.io/kaniko-project/executor:v1.14.0-debug.
    entrypoint: [""]  # Устанавливает точку входа (entrypoint) для образа Docker. В данном случае, точка входа не определена, поэтому будет использоваться точка входа, заданная в самом образе.
  script:             # Определяет команды, которые будут выполнены внутри контейнера на базе указанного образа.
    - /kaniko/executor
      --context "${CI_PROJECT_DIR}"
      --dockerfile "${CI_PROJECT_DIR}/Dockerfile"
      --insecure
      --destination "${CI_REGISTRY_IMAGE}:${CI_COMMIT_TAG}"
    # /kaniko/executor                                      //Запускает команду kaniko, инструмент для сборки Docker-образов из Dockerfile без необходимости доступа к Docker демону.
    # --context "${CI_PROJECT_DIR}"                         //Задает контекст сборки, который равен корневой директории проекта ${CI_PROJECT_DIR}.
    # --dockerfile "${CI_PROJECT_DIR}/Dockerfile"           //Указывает путь к Dockerfile, который будет использован для сборки образа.
    # --insecure                                            // Включает режим небезопасной сборки, что позволяет использовать HTTP вместо HTTPS для загрузки образов.
    # --destination "${CI_REGISTRY_IMAGE}:${CI_COMMIT_TAG}" // Указывает место назначения (destination) для собранного Docker-образа. В данном случае, образ будет загружен в GitLab Container Registry с тегом ${CI_COMMIT_TAG}.
  rules:                  # Определяет правила, при выполнении которых будет запущена задача
    - if: $CI_COMMIT_TAG  #  Указывает, что задача build будет выполняться только в случае наличия переменной CI_COMMIT_TAG. В нашем случае задача собирает и публикует Docker-образ при создании нового тега.
```
Успешные статусы выполнения пайплайна:
![Успешные статусы выполнения пайплайна](/assets/pic/pipeline_docker_build.png)

Лог успешного выполнения пайплайна:
![Лог успешного выполнения пайплайна](/assets/pic/docker_build_log.png)


Образы находятся в Gitlab Container Registry:
![image находится в registry](/assets/pic/image_registry.png)

## Настройки отправки изменений из gitlab в gitlab
Для этих целей используем встроенный функционал GitLab зеркалирования реп:
![Настройки отправки изменений из gitlab в gitlab](/assets/pic/mirror_to_gitlab2.png)

> [!Замечание]
> В репозитории gitlab2 для зеркалирования отключен CI/CD.

Успешный синк выглядит примерно так:
![Успешный синк репы](/assets/picmirror_to_gitlab2_status2.png)

## На втором gitlab’е необходимо построить pipeline так, чтобы при появлении изменений запускался любой sast анализ изменений.
Для этих целей создан новый репозиторий с триггером:
![Триггер](/assets/pic/trigger.png)

В репозитории, в котором хранится зеркальная копия включен webhook:
![Webhook](/assets/pic/webhook.png)

Разрешаем обращения в локальной сети для webhook`ов:
![Разрешаем обращения в локальной сети для webhookов](/assets/pic/allow_local_network.png)

И настроен пайплайн:
```
$ cat .gitlab-ci.yml
stages:   # Определяет этапы (stages) выполнения пайплайна CI/CD.
  - sast  # В данном случае, есть только один этап - sast.

variables:  # Определяет переменные окружения, которые будут доступны во всех задачах (jobs) пайплайна.
  SONAR_USER_HOME: "${CI_PROJECT_DIR}/.sonar" # Определяет расположение кэша задачи анализа. Кэш используется для оптимизации повторных запусков анализа.
  GIT_DEPTH: "0"                              # Устанавливает глубину клонирования Git-репозитория в 0. Это гарантирует, что все ветки проекта будут получены.
  TARGET_REPO_URL: "http://oauth2:${TARGET_REPO_ACCESS_TOKEN}@${CI_SERVER_HOST}:${CI_SERVER_PORT}/user2/mirror-demo-image.git" # Определяет URL репозитория, который будет анализироваться. В данном случае, используется переменная $TARGET_REPO_ACCESS_TOKEN, а также встроенные переменные $CI_SERVER_HOST и $CI_SERVER_PORT, чтобы сформировать URL.
  TARGET_DIR: "scan_repo"                     # Определяет имя директории, в которой будет склонирован репозиторий для анализа.

sonarqube-check:  # Определяет задачу (job) с именем sonarqube-check, которая выполняет статический анализ кода с помощью SonarQube.
  stage: sast     # Указывает, что задача sonarqube-check принадлежит этапу sast.
  image:          # Определяет Docker-образ, который будет использоваться для выполнения задачи.
    name: sonarsource/sonar-scanner-cli:5.0.1   # В данном случае, используется образ sonar-scanner-cli c тегом 5.0.1.
    entrypoint: [""]                            # Устанавливает точку входа (entrypoint) для образа. В данном случае, точка входа не определена, поэтому будет использоваться точка входа, заданная в самом образе.
  cache:                  # Настраивает кэширование для оптимизации повторных запусков анализа.
    key: "${CI_JOB_NAME}" # Определяет ключ (key) для кэша. Здесь используется переменная ${CI_JOB_NAME}, которая представляет собой имя текущей задачи CI/CD. Каждая задача будет иметь свой уникальный ключ кэша на основе имени задачи. Это гарантирует, что кэши будут отдельными для разных задач.
    paths:                # Определяет пути (paths), которые будут кэшироваться.
      - .sonar/cache      # В данном случае, указывается `.sonar/cache`.
  script:                 # Определяет команды, которые будут выполнены внутри контейнера на базе указанного образа.
    - git clone ${TARGET_REPO_URL} ${TARGET_DIR}        # Клонирует репозиторий с заданным URL в директорию ${TARGET_DIR}.
    - sonar-scanner -Dsonar.sources=${TARGET_DIR}/src   # Запускает Sonar Scanner для выполнения анализа кода и указывается исходная директория для анализа.
```
```
$ cat sonar-project.properties
#  Устанавливаем уникальный ключ проекта в SonarQube. Ключ проекта служит для идентификации проекта в системе и используется при отправке результатов анализа на сервер SonarQube
sonar.projectKey=Demo-image
# Указываем организацию (группу проектов) в SonarQube, к которой принадлежит проект. Она используется в многопроектных средах, чтобы организовать проекты внутри одной организации.
sonar.organization=X

# Отключаем исключения, связанные с системой управления версиями (scm)
sonar.scm.exclusions.disabled=true

# Указываем версии Python для анализа
sonar.python.version=2.7, 3.7, 3.8, 3.9
```

Успешные статусы пайплайна:
![Успешные статусы пайплайна](/assets/pic/pipline_run_sast.png)

Лог успешного пайплайна:
```
Running with gitlab-runner 16.4.0 (6e766faf)
  on My Docker Runner siQAbyvN, system ID: r_lE13R13naV7g
Preparing the "docker" executor
00:03
Using Docker executor with image sonarsource/sonar-scanner-cli:5.0.1 ...
Pulling docker image sonarsource/sonar-scanner-cli:5.0.1 ...
Using docker image sha256:2f384fb1bbd5f033fa0b628efb5ef3d40b9cafaddb68b9ffdd8c3cacdc237199 for sonarsource/sonar-scanner-cli:5.0.1 with digest sonarsource/sonar-scanner-cli@sha256:494ecc3b5b1ee1625bd377b3905c4284e4f0cc155cff397805a244dee1c7d575 ...
Preparing environment
00:01
Running on runner-siqabyvn-project-2-concurrent-0 via 130107f49988...
Getting source from Git repository
00:00
Fetching changes...
Reinitialized existing Git repository in /builds/user2/sast-check/.git/
Checking out 559a31de as detached HEAD (ref is main)...
Removing .scannerwork/
Removing .sonar/
Removing scan_repo/
Skipping Git submodules setup
Restoring cache
00:03
Checking cache for sonarqube-check-protected...
No URL provided, cache will not be downloaded from shared cache server. Instead a local version of cache will be extracted. 
Successfully extracted cache
Executing "step_script" stage of the job script
00:05
Using docker image sha256:2f384fb1bbd5f033fa0b628efb5ef3d40b9cafaddb68b9ffdd8c3cacdc237199 for sonarsource/sonar-scanner-cli:5.0.1 with digest sonarsource/sonar-scanner-cli@sha256:494ecc3b5b1ee1625bd377b3905c4284e4f0cc155cff397805a244dee1c7d575 ...
$ git clone ${TARGET_REPO_URL} ${TARGET_DIR}
Cloning into 'scan_repo'...
$ sonar-scanner -Dsonar.sources=${TARGET_DIR}/src
INFO: Scanner configuration file: /opt/sonar-scanner/conf/sonar-scanner.properties
INFO: Project root configuration file: /builds/user2/sast-check/sonar-project.properties
INFO: SonarScanner 5.0.1.3006
INFO: Java 17.0.8 Alpine (64-bit)
INFO: Linux 6.2.0-34-generic amd64
INFO: User cache: /builds/user2/sast-check/.sonar/cache
INFO: Analyzing on SonarQube server 9.9.2.77730
INFO: Default locale: "en_US", source code encoding: "UTF-8" (analysis is platform dependent)
INFO: Load global settings
INFO: Load global settings (done) | time=48ms
INFO: Server id: 147B411E-AYsA73xoB-OxWGT5kjS8
INFO: User cache: /builds/user2/sast-check/.sonar/cache
INFO: Load/download plugins
INFO: Load plugins index
INFO: Load plugins index (done) | time=25ms
INFO: Load/download plugins (done) | time=78ms
INFO: Process project properties
INFO: Process project properties (done) | time=6ms
INFO: Execute project builders
INFO: Execute project builders (done) | time=2ms
INFO: Project key: Demo-image
INFO: Base dir: /builds/user2/sast-check
INFO: Working dir: /builds/user2/sast-check/.scannerwork
INFO: Load project settings for component key: 'Demo-image'
INFO: Load project settings for component key: 'Demo-image' (done) | time=13ms
INFO: Exclusions based on SCM info is disabled by configuration
INFO: Auto-configuring with CI 'Gitlab CI'
INFO: Load quality profiles
INFO: Load quality profiles (done) | time=25ms
INFO: Load active rules
INFO: Load active rules (done) | time=837ms
INFO: Load analysis cache
INFO: Load analysis cache (269 bytes) | time=8ms
INFO: Load project repositories
INFO: Load project repositories (done) | time=9ms
INFO: Indexing files...
INFO: Project configuration:
INFO: 1 file indexed
INFO: Quality profile for py: Sonar way
INFO: ------------- Run sensors on module Demo-image
INFO: Load metrics repository
INFO: Load metrics repository (done) | time=14ms
INFO: Sensor Python Sensor [python]
INFO: Starting global symbols computation
INFO: 1 source file to be analyzed
INFO: 1/1 source file has been analyzed
INFO: Starting rules execution
INFO: 1 source file to be analyzed
INFO: 1/1 source file has been analyzed
INFO: The Python analyzer was able to leverage cached data from previous analyses for 0 out of 1 files. These files were not parsed.
INFO: Sensor Python Sensor [python] (done) | time=273ms
INFO: Sensor Cobertura Sensor for Python coverage [python]
INFO: Sensor Cobertura Sensor for Python coverage [python] (done) | time=13ms
INFO: Sensor PythonXUnitSensor [python]
INFO: Sensor PythonXUnitSensor [python] (done) | time=10ms
INFO: Sensor JaCoCo XML Report Importer [jacoco]
INFO: 'sonar.coverage.jacoco.xmlReportPaths' is not defined. Using default locations: target/site/jacoco/jacoco.xml,target/site/jacoco-it/jacoco.xml,build/reports/jacoco/test/jacocoTestReport.xml
INFO: No report imported, no coverage information will be imported by JaCoCo XML Report Importer
INFO: Sensor JaCoCo XML Report Importer [jacoco] (done) | time=2ms
INFO: Sensor CSS Rules [javascript]
INFO: No CSS, PHP, HTML or VueJS files are found in the project. CSS analysis is skipped.
INFO: Sensor CSS Rules [javascript] (done) | time=0ms
INFO: Sensor C# Project Type Information [csharp]
INFO: Sensor C# Project Type Information [csharp] (done) | time=1ms
INFO: Sensor C# Analysis Log [csharp]
INFO: Sensor C# Analysis Log [csharp] (done) | time=8ms
INFO: Sensor C# Properties [csharp]
INFO: Sensor C# Properties [csharp] (done) | time=0ms
INFO: Sensor HTML [web]
INFO: Sensor HTML [web] (done) | time=1ms
INFO: Sensor TextAndSecretsSensor [text]
INFO: 1 source file to be analyzed
INFO: 1/1 source file has been analyzed
INFO: Sensor TextAndSecretsSensor [text] (done) | time=7ms
INFO: Sensor VB.NET Project Type Information [vbnet]
INFO: Sensor VB.NET Project Type Information [vbnet] (done) | time=1ms
INFO: Sensor VB.NET Analysis Log [vbnet]
INFO: Sensor VB.NET Analysis Log [vbnet] (done) | time=7ms
INFO: Sensor VB.NET Properties [vbnet]
INFO: Sensor VB.NET Properties [vbnet] (done) | time=0ms
INFO: Sensor IaC Docker Sensor [iac]
INFO: 0 source files to be analyzed
INFO: 0/0 source files have been analyzed
INFO: Sensor IaC Docker Sensor [iac] (done) | time=44ms
INFO: ------------- Run sensors on project
INFO: Sensor Analysis Warnings import [csharp]
INFO: Sensor Analysis Warnings import [csharp] (done) | time=0ms
INFO: Sensor Zero Coverage Sensor
INFO: Sensor Zero Coverage Sensor (done) | time=3ms
INFO: SCM Publisher SCM provider for this project is: git
INFO: SCM Publisher 1 source file to be analyzed
INFO: SCM Publisher 0/1 source files have been analyzed (done) | time=166ms
WARN: Missing blame information for the following files:
WARN:   * scan_repo/src/app.py
WARN: This may lead to missing/broken features in SonarQube
INFO: CPD Executor 1 file had no CPD blocks
INFO: CPD Executor Calculating CPD for 0 files
INFO: CPD Executor CPD calculation finished (done) | time=0ms
INFO: Analysis report generated in 42ms, dir size=122.5 kB
INFO: Analysis report compressed in 9ms, zip size=16.4 kB
INFO: Analysis report uploaded in 53ms
INFO: ANALYSIS SUCCESSFUL, you can find the results at: http://sonarqube.xxx.trash:9000/dashboard?id=Demo-image
INFO: Note that you will be able to access the updated dashboard once the server has processed the submitted analysis report
INFO: More about the report processing at http://sonarqube.xxx.trash:9000/api/ce/task?id=AYsBA_1WB-OxWGT5ktMa
INFO: Analysis total time: 3.133 s
INFO: ------------------------------------------------------------------------
INFO: EXECUTION SUCCESS
INFO: ------------------------------------------------------------------------
INFO: Total time: 3.849s
INFO: Final Memory: 23M/120M
INFO: ------------------------------------------------------------------------
Saving cache for successful job
00:00
Creating cache sonarqube-check-protected...
.sonar/cache: found 55 matching artifact files and directories 
Archive is up to date!                             
Created cache
Cleaning up project directory and file based variables
00:01
Job succeeded
```
