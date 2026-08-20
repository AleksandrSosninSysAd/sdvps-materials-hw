 
# Домашнее задание 8.2. Что такое DevOps. CI/CD

## Задание 1: Настройка Jenkins и создание Freestyle Project

### Что было сделано:

1. **Установлен Jenkins** на Ubuntu 24.04
2. **Установлен Go** версии 1.21.0
3. **Установлен Docker** и пользователь jenkins добавлен в группу docker
4. **Сделан форк** репозитория: https://github.com/AleksandrSosninSysAd/sdvps-materials-hw
5. **Создан Freestyle Project** в Jenkins с настройками:
   - Repository URL: https://github.com/AleksandrSosninSysAd/sdvps-materials-hw.git
   - Branch: */main
   - Build steps: Execute shell

### Скрипт сборки:

```bash
export PATH=$PATH:/usr/local/go/bin
go version
go test .
docker build . -t my-hello-world:v1
```


---

## Скриншоты

### Настройки проекта (Configure)
![Настройки проекта](screenshots/configure.png)

### Console Output
![Console Output](screenshots/console-output.png)

---

## Задание 2: Настройка Declarative Pipeline

### Что было сделано:
1. Создан новый проект типа **Pipeline** в Jenkins
2. Сборка из Задания 1 переписана на декларативный синтаксис (Declarative Pipeline)
3. Настроены этапы:
   - **Git** — клонирование репозитория
   - **Test** — запуск `go test .`
   - **Build** — сборка Docker-образа с тегом `v$BUILD_NUMBER`

### Скрипт Pipeline:
```groovy
pipeline {
    agent any
    
    stages {
        stage('Git') {
            steps {
                git branch: 'main', url: 'https://github.com/AleksandrSosninSysAd/sdvps-materials-hw.git'
            }
        }
        stage('Test') {
            steps {
                sh '''
                    export PATH=$PATH:/usr/local/go/bin
                    go version
                    go test .
                '''
            }
        }
        stage('Build') {
            steps {
                sh 'docker build . -t my-hello-world:v$BUILD_NUMBER'
            }
        }
    }
}
```
---

## Скриншоты

### Настройки проекта (Configure)
![Настройки проекта](screenshots/configure2.png)

### Console Output
![Console Output](screenshots/console-output2.png)

### Main
![Main Page](screenshots/main-page2.png)

---
---

## Задание 3: Установка Nexus и загрузка бинарного файла

### Что было сделано:
1. **Установлен Nexus** в Docker контейнере (порт 8081 для UI, 8082 для Docker registry)
2. **Создан raw-hosted репозиторий** `my-releases` в Nexus
3. **Изменен Pipeline** в Jenkins:
   - Вместо сборки Docker-образа собирается бинарный Go-файл (`myapp`)
   - Бинарный файл загружается в Nexus через `curl`
4. **Настроены Credentials** в Jenkins для авторизации в Nexus

### Скрипт Pipeline:
```groovy
pipeline {
    agent any
    
    stages {
        stage('Git') {
            steps {
                git branch: 'main', url: 'https://github.com/AleksandrSosninSysAd/sdvps-materials-hw.git'
            }
        }
        
        stage('Test') {
            steps {
                sh '''
                    export PATH=$PATH:/usr/local/go/bin
                    go version
                    go test .
                '''
            }
        }
        
        stage('Build Binary') {
            steps {
                sh '''
                    export PATH=$PATH:/usr/local/go/bin
                    CGO_ENABLED=0 GOOS=linux go build -a -installsuffix nocgo -o myapp .
                    ls -la myapp
                '''
            }
        }
        
        stage('Upload to Nexus') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'nexus-credentials', passwordVariable: 'NEXUS_PASS', usernameVariable: 'NEXUS_USER')]) {
                        sh '''
                            curl -v -u $NEXUS_USER:$NEXUS_PASS \
                            --upload-file myapp \
                            http://127.0.0.1:8081/repository/my-releases/myapp-$BUILD_NUMBER
                        '''
                    }
                }
            }
        }
    }
}
```
---

## Скриншоты

### Настройки проекта (Configure)
![Pipeline Configure Task 3](screenshots/nexus-pipeline-configure.png)

### Console Output
![Console Output Task 3](screenshots/nexus-console-output.png)

### Main
![Nexus my-releases](screenshots/nexus-my-releases.png)

---

