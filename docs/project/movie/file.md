---
comments: true
---
# Triển khai service file lên k8s
## 1. Giới thiệu

Bài viết này giới thiệu cách triển trai thử nghiệm api service file lên k8s.

- Source code: [https://github.com/vanphuoc9/file.git](https://github.com/vanphuoc9/file.git) 
- Source file config: [https://github.com/vanphuoc9/file-config.git](https://github.com/vanphuoc9/file-config.git)

Bài viết này giới thiệu cách triển khai một service lên k8s bằng jenkins và helm.

Các bài viết tiếp theo mình sẽ giới thiệu cách setup các service và web app khi khởi tạo hệ thống ban đầu bằng 1 file helm duy nhất chong các service.

## 2. Cài đặt
### 2.1. Dùng jenkins build dockerfile và sau đó push image lên docker hub
Trong project file [https://github.com/vanphuoc9/file.git](https://github.com/vanphuoc9/file.git) có file:

Dockerfile: dùng để cấu hình build source code thành file jar

```bash title="Dockerfile"  linenums="1"
# Stage 1: Build the application using Gradle
FROM gradle:8.7-jdk21 AS builder

# Copy everything and build the app
WORKDIR /home/gradle/project
COPY --chown=gradle:gradle . .

# Build the app (using the bootJar task)
RUN gradle bootJar --no-daemon

# Stage 2: Run the application using a minimal JDK runtime
FROM eclipse-temurin:21-jdk-jammy

WORKDIR /app

# Copy the fat JAR from the builder stage
COPY --from=builder /home/gradle/project/build/libs/*.jar app.jar

# Expose the port (matching server.port=9081)
EXPOSE 9081

# Run the Spring Boot app
ENTRYPOINT ["java", "-jar", "app.jar"]

```


Jenkinsfile-k8s-full: dùng jenkins để build Dockerfile ở trên thành image và đẩy image đó lên dockerhub

```bash title="Jenkinsfile-k8s-full"  linenums="1"
def appConfigRepo = 'https://github.com/vanphuoc9/file-config.git'
def appConfigBranch = 'main'

def helmRepo = "file-config"
def helmValueFile = "values.yaml"

pipeline {
    agent any

    environment {
        BUILD_NUMBER = "${env.BUILD_NUMBER}"
        version = "v1.${BUILD_NUMBER}"
        DOCKER_IMAGE_NAME = "thaiphuoc1997/svcfile"
        DOCKER_IMAGE = "${DOCKER_IMAGE_NAME}:${version}"
    }

    options {
        skipDefaultCheckout()
    }

    parameters {
        string(name: 'GIT_URL', defaultValue: 'https://github.com/vanphuoc9/file.git', description: 'The URL of the source Git repository to use.')
        string(name: 'GIT_BRANCH', defaultValue: 'master' , description: 'The branch in the source Git repository to use.')
    }

    stages {
        stage("Checkout") {
            steps {
                checkout(changelog: false, poll: false, scm: [
                    $class: 'GitSCM',
                    branches: [[name: params.GIT_BRANCH]],
                    doGenerateSubmoduleConfigurations: false,
                    submoduleCfg: [],
                    userRemoteConfigs: [[url: params.GIT_URL]],
                ])
                stash name: 'sources', includes: '**', excludes: '**/.git,**/.git/**'
            }
        }

        stage("Build And Push Docker Image") {
            agent {
                label 'docker-build'
            }
            steps {
                unstash 'sources'
                container(name: 'kaniko') {
                    sh '/kaniko/executor --context=`pwd` --dockerfile=`pwd`/Dockerfile --destination=${DOCKER_IMAGE}'
                }
            }
        }

        // stage('Update version in helm-chart') {
        //     steps {
        //         withCredentials([usernamePassword(credentialsId: 'github', passwordVariable: 'GIT_PASSWORD', usernameVariable: 'GIT_USERNAME')]) {
        //             sh """#!/bin/bash
        //                 set -e

        //                 [[ -d ${helmRepo} ]] && rm -rf ${helmRepo}

        //                 git clone ${appConfigRepo} --branch ${appConfigBranch}
        //                 cd ${helmRepo}

        //                 # Cập nhật tag với version
        //                 sed -i "s|  tag: .*|  tag: \\\"${version}\\\"|" ${helmValueFile}

        //                 git config user.name "vanphuoc9"
        //                 git config user.email "thaiphuoc1997@gmail.com"

        //                 git add .
        //                 git commit -m "Update to version ${version}" || echo "No changes to commit"
        //                 git push https://${GIT_USERNAME}:${GIT_PASSWORD}@github.com/vanphuoc9/file-config.git

        //                 cd ..
        //                 rm -rf ${helmRepo}
        //             """
        //         }
        //     }
        // }
    }
}

```

Có thể thực hiện theo các bước ở các bài viết trước: [Cấu hình luồng Jenkins Build Docker image và push lên Docker Hub](/Kubernetes%20(K8s)/Gitops/jenkins-build-image)

### 2.2. Tạo helm và triển khai lên k8s

Tạo helm file-config


```bash title="application.properties"  linenums="1"
helm create file-config

```
Sau khi tạo helm xong thì bài toán đặt ra làm cách nào cấu hình các tham số của spring boot lên k8s, do có thể cấu hình các môi trường khác nhau như production, testing, dev thì cấu hình lại khác nhau


Ví dụ trong application.properties của spring boot có các tham số môi trường như sau:

```bash title="application.properties"  linenums="1"
spring.application.name=file

server.port=9081

minio.url=${MINIO_URL:http://192.168.1.100:30012}
minio.bucket=${MINIO_BUCKET:movie}
minio.accessKey=${MINIO_ACCESS_KEY:8ADlXpiyA33VRayeDqGn}
minio.secretKey=${MINIO_SECRET_KEY:HnZTWCxRdLqGHonzvL1WYgQ5Oej2O2ShT29Hb2pv}


#spring.security.oauth2.resourceserver.jwt.issuer-uri=${ISSUER_URI:http://auth-production.reb.com/realms/reb} # tam thoi tat de test deploy tren k8s
spring.security.oauth2.resourceserver.jwt.jwk-set-uri=${JWK_SET_URI:http://auth-production.reb.com/realms/reb/protocol/openid-connect/certs}

spring.data.mongodb.uri=${MONGODB_URI:mongodb://rebdb:rebdb@192.168.1.100:30011/rebdb?retryWrites=true&w=majority}

#Servlet Multipart Properties
spring.servlet.multipart.enabled=${MULTIPART_ENABLE:true}
spring.servlet.multipart.max-file-size=${MAX_FILE_SIZE:500MB}
spring.servlet.multipart.max-request-size=${MAX_REQUEST_SIZE:500MB}
spring.servlet.multipart.file-size-threshold=${FILE_SIZE_THRESHOLD:2KB}


springdoc.swagger-ui.enabled=${STRING_DOC_SWAGGER_UI_ENABLE:true}
springdoc.api-docs.enabled=${STRING_DOC_API_DOCS_ENABLE:true}
springdoc.api-docs.path=${STRING_DOC_API_DOCS_PATH:/api-docs}
springdoc.swagger-ui.path=${STRING_DOC_SWAGGER_UI_PATH://swagger-ui.html}

```

Khi deploy lên k8s cần thay các tham số cấu hình thì chúng ta làm như sau:

Tạo file configmap để cấu hình các thông tin không nhạy cảm

```yaml title="configmap.yaml"  linenums="1"
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ include "file-config.fullname" . }}-config
  labels:
    {{- include "file-config.labels" . | nindent 4 }}
data:
  SPRING_APPLICATION_NAME: {{ .Values.config.springApplicationName | quote }}
  SERVER_PORT: {{ .Values.config.serverPort | quote }}
  SPRING_SERVLET_MULTIPART_ENABLED: {{ .Values.config.multipartEnabled | quote }}
  SPRING_SERVLET_MULTIPART_MAX_FILE_SIZE: {{ .Values.config.maxFileSize | quote }}
  SPRING_SERVLET_MULTIPART_MAX_REQUEST_SIZE: {{ .Values.config.maxRequestSize | quote }}
  SPRING_SERVLET_MULTIPART_FILE_SIZE_THRESHOLD: {{ .Values.config.fileSizeThreshold | quote }}
  SPRINGDOC_SWAGGER_UI_ENABLED: {{ .Values.config.springdocSwaggerUiEnabled | quote }}
  SPRINGDOC_API_DOCS_ENABLED: {{ .Values.config.springdocApiDocsEnabled | quote }}
  SPRINGDOC_API_DOCS_PATH: {{ .Values.config.springdocApiDocsPath | quote }}
  SPRINGDOC_SWAGGER_UI_PATH: {{ .Values.config.springdocSwaggerUiPath | quote }}

```

Tạo file secret để cấu hình các thông tin nhạy cảm:

```yaml title="secret.yaml"  linenums="1"
apiVersion: v1
kind: Secret
metadata:
  name: {{ include "file-config.fullname" . }}-secret
  labels:
    {{- include "file-config.labels" . | nindent 4 }}
type: Opaque
data:
  MINIO_URL: {{ .Values.secret.minioUrl | b64enc }}
  MINIO_BUCKET: {{ .Values.secret.minioBucket | b64enc }}
  MINIO_ACCESS_KEY: {{ .Values.secret.minioAccessKey | b64enc }}
  MINIO_SECRET_KEY: {{ .Values.secret.minioSecretKey | b64enc }}
  ISSUER_URI: {{ .Values.secret.issuerUri | b64enc }}
  JWK_SET_URI: {{ .Values.secret.jwkSetUri | b64enc }}
  MONGODB_URI: {{ .Values.secret.mongodbUri | b64enc }}
```

 Trong file values.yaml cấu hình các thông số như phiên bản image, các thông số cấu hình,...


```yaml title="values.yaml"  linenums="1"
# Default values for file-config.
# This is a YAML-formatted file.
# Declare variables to be passed into your templates.

# This will set the replicaset count more information can be found here: https://kubernetes.io/docs/concepts/workloads/controllers/replicaset/
replicaCount: 1

# This sets the container image more information can be found here: https://kubernetes.io/docs/concepts/containers/images/
image:
  repository: thaiphuoc1997/svcfile
  # This sets the pull policy for images.
  pullPolicy: IfNotPresent
  # Overrides the image tag whose default is the chart appVersion.
  tag: "v1.7"

# This is for the secretes for pulling an image from a private repository more information can be found here: https://kubernetes.io/docs/tasks/configure-pod-container/pull-image-private-registry/
imagePullSecrets: []
# This is to override the chart name.
nameOverride: ""
fullnameOverride: ""

#This section builds out the service account more information can be found here: https://kubernetes.io/docs/concepts/security/service-accounts/
serviceAccount:
  # Specifies whether a service account should be created
  create: true
  # Automatically mount a ServiceAccount's API credentials?
  automount: true
  # Annotations to add to the service account
  annotations: {}
  # The name of the service account to use.
  # If not set and create is true, a name is generated using the fullname template
  name: ""

# This is for setting Kubernetes Annotations to a Pod.
# For more information checkout: https://kubernetes.io/docs/concepts/overview/working-with-objects/annotations/ 
podAnnotations: {}
# This is for setting Kubernetes Labels to a Pod.
# For more information checkout: https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/
podLabels: {}

podSecurityContext: {}
  # fsGroup: 2000

securityContext: {}
  # capabilities:
  #   drop:
  #   - ALL
  # readOnlyRootFilesystem: true
  # runAsNonRoot: true
  # runAsUser: 1000

# This is for setting up a service more information can be found here: https://kubernetes.io/docs/concepts/services-networking/service/
service:
  # This sets the service type more information can be found here: https://kubernetes.io/docs/concepts/services-networking/service/#publishing-services-service-types
  type: ClusterIP
  # This sets the ports more information can be found here: https://kubernetes.io/docs/concepts/services-networking/service/#field-spec-ports
  port: 9081

# This block is for setting up the ingress for more information can be found here: https://kubernetes.io/docs/concepts/services-networking/ingress/
ingress:
  enabled: true
  className: ""
  annotations: 
    kubernetes.io/ingress.class: nginx
    # kubernetes.io/tls-acme: "true"
  hosts:
    - host: filesvc.reb.com
      paths:
        - path: /
          pathType: Prefix
  tls: []
  #  - secretName: chart-example-tls
  #    hosts:
  #      - chart-example.local

resources: {}
  # We usually recommend not to specify default resources and to leave this as a conscious
  # choice for the user. This also increases chances charts run on environments with little
  # resources, such as Minikube. If you do want to specify resources, uncomment the following
  # lines, adjust them as necessary, and remove the curly braces after 'resources:'.
  # limits:
  #   cpu: 100m
  #   memory: 128Mi
  # requests:
  #   cpu: 100m
  #   memory: 128Mi

# Non-sensitive configuration (ConfigMap)
config:
  springApplicationName: "file"
  serverPort: 9081
  multipartEnabled: true
  maxFileSize: "500MB"
  maxRequestSize: "500MB"
  fileSizeThreshold: "2KB"
  springdocSwaggerUiEnabled: true
  springdocApiDocsEnabled: true
  springdocApiDocsPath: "/api-docs"
  springdocSwaggerUiPath: "/swagger-ui.html"

# Sensitive configuration (Secret)
secret:
  minioUrl: "http://minio-service:9000"
  minioBucket: "movie"
  minioAccessKey: "8ADlXpiyA33VRayeDqGn"
  minioSecretKey: "HnZTWCxRdLqGHonzvL1WYgQ5Oej2O2ShT29Hb2pv"
  issuerUri: "http://keycloak-service:8080/realms/reb"
  jwkSetUri: "http://keycloak-service:8080/realms/reb/protocol/openid-connect/certs"
  mongodbUri: "mongodb://rebdb:rebdb@mongo-service:27017/rebdb?retryWrites=true&w=majority"

# Environment variables
env:
  SPRING_PROFILES_ACTIVE: prod

# This is to setup the liveness and readiness probes more information can be found here: https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/
livenessProbe:
  httpGet:
    path: /actuator/health
    port: 9081
readinessProbe:
  httpGet:
    path: /actuator/health
    port: 9081

#This section is for setting up autoscaling more information can be found here: https://kubernetes.io/docs/concepts/workloads/autoscaling/
autoscaling:
  enabled: false
  minReplicas: 1
  maxReplicas: 100
  targetCPUUtilizationPercentage: 80
  # targetMemoryUtilizationPercentage: 80

# Additional volumes on the output Deployment definition.
volumes: []
# - name: foo
#   secret:
#     secretName: mysecret
#     optional: false

# Additional volumeMounts on the output Deployment definition.
volumeMounts: []
# - name: foo
#   mountPath: "/etc/foo"
#   readOnly: true

nodeSelector: {}

tolerations: []

affinity: {}

```

!!! note "Lưu ý"

    jwkSetUri: "http://keycloak-service:8080/realms/reb/protocol/openid-connect/certs" => do domain http://auth-production.reb.com chưa public nên mình dùng http://keycloak-service:8080 để kiểm tra auth api, chỗ này domain public rồi thì nên dùng domain http://auth-production.reb.com

Vào thư mục chứa helm của service file trong server, chạy lệnh sau:

```bash 
helm upgrade --install svcfile . --namespace dev --set image.tag=v1.7
```

!!! note "Lưu ý"

    Có thể đổi cấu hình khác trong helm để phù hợp với môi trường mình cài đặt


Muốn xóa helm vừa tạo thì

```bash 
helm uninstall svcfile -n dev
```

### 2.3. Triển khai toàn trình với jenkins

Sau khi chạy helm ổn rồi thì tiến hành bỏ comment "Update version in helm-chart", và cấu hình Argocd để triển khai tự động lên K8S

```bash title="Jenkinsfile-k8s-full"  linenums="1"
def appConfigRepo = 'https://github.com/vanphuoc9/file-config.git'
def appConfigBranch = 'main'

def helmRepo = "file-config"
def helmValueFile = "values.yaml"

pipeline {
    agent any

    environment {
        BUILD_NUMBER = "${env.BUILD_NUMBER}"
        version = "v1.${BUILD_NUMBER}"
        DOCKER_IMAGE_NAME = "thaiphuoc1997/svcfile"
        DOCKER_IMAGE = "${DOCKER_IMAGE_NAME}:${version}"
    }

    options {
        skipDefaultCheckout()
    }

    parameters {
        string(name: 'GIT_URL', defaultValue: 'https://github.com/vanphuoc9/file.git', description: 'The URL of the source Git repository to use.')
        string(name: 'GIT_BRANCH', defaultValue: 'master' , description: 'The branch in the source Git repository to use.')
    }

    stages {
        stage("Checkout") {
            steps {
                checkout(changelog: false, poll: false, scm: [
                    $class: 'GitSCM',
                    branches: [[name: params.GIT_BRANCH]],
                    doGenerateSubmoduleConfigurations: false,
                    submoduleCfg: [],
                    userRemoteConfigs: [[url: params.GIT_URL]],
                ])
                stash name: 'sources', includes: '**', excludes: '**/.git,**/.git/**'
            }
        }

        stage("Build And Push Docker Image") {
            agent {
                label 'docker-build'
            }
            steps {
                unstash 'sources'
                container(name: 'kaniko') {
                    sh '/kaniko/executor --context=`pwd` --dockerfile=`pwd`/Dockerfile --destination=${DOCKER_IMAGE}'
                }
            }
        }

        stage('Update version in helm-chart') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'github', passwordVariable: 'GIT_PASSWORD', usernameVariable: 'GIT_USERNAME')]) {
                    sh """#!/bin/bash
                        set -e

                        [[ -d ${helmRepo} ]] && rm -rf ${helmRepo}

                        git clone ${appConfigRepo} --branch ${appConfigBranch}
                        cd ${helmRepo}

                        # Cập nhật tag với version
                        sed -i "s|  tag: .*|  tag: \\\"${version}\\\"|" ${helmValueFile}

                        git config user.name "vanphuoc9"
                        git config user.email "thaiphuoc1997@gmail.com"

                        git add .
                        git commit -m "Update to version ${version}" || echo "No changes to commit"
                        git push https://${GIT_USERNAME}:${GIT_PASSWORD}@github.com/vanphuoc9/file-config.git

                        cd ..
                        rm -rf ${helmRepo}
                    """
                }
            }
        }
    }
}

```

Sau đó cấu hình theo bài viết: [Toàn luồng Jenkins + Argocd + Github + Docker hub trên k8s](/Kubernetes%20(K8s)/Gitops/full-jenkins-argocd)
