pipeline {
  agent none
  stages {
    stage('Docker Build') {
      matrix {
        agent any
        axes {
          axis {
            name 'PLATFORM'
            values 'windows-2019'
          }
          axis {
            name 'JDK_VERSION'
            values '8', '11', '14', '15'
          }
          axis {
            name 'JDK_TYPE'
            values 'hotspot', 'openj9'
          }
          axis {
            name 'TYPE'
            values 'jdk', 'jre'
          }
        }
        stages {
          stage('Build') {
            agent {
              label "amd64&&windock&&windows"
            }
            stages {
              stage("build") {
                environment {
                    DOCKER_FILE = ".\\${JDK_VERSION}\\${TYPE}\\windows\\windowsservercore-ltsc2019\\Dockerfile.${JDK_TYPE}.releases.full"
                    FULL_JDK_VERSION = getJavaVersion(env.DOCKER_FILE)
                    TAG_STRING = "-t ${getTags(JDK_VERSION, FULL_JDK_VERSION, TYPE, JDK_TYPE).join(' -t ')}"
                }
                steps { 
                  script {
                    publishChecks name: "${JDK_VERSION} / ${JDK_TYPE} / ${TYPE}", title: 'Docker Build', status: "IN_PROGRESS"
                    echo "Do Build for ${PLATFORM} / ${JDK_VERSION} / ${JDK_TYPE} / ${TYPE}"
                    echo env.TAG_STRING
                  
                    bat "docker build -f ${env.DOCKER_FILE} ${env.TAG_STRING} c:\\temp\\"
                    infra.withDockerCredentials {
                      getTags(JDK_VERSION, env.FULL_JDK_VERSION, TYPE, JDK_TYPE).each{ tag -> 
                        bat "docker push ${tag}"
                      }
                    }
                    publishChecks name: "${JDK_VERSION} / ${JDK_TYPE} / ${TYPE}", title: 'Docker Build', status: "COMPLETED"
                  }  
                }
              }
            }
          }
        }
      }
    }
  }
}

def getJavaVersion(path) {
  def contents = readFile file: path
  def javaVersion = contents.split('\n').find{ it.startsWith('ENV JAVA_VERSION') }.replace('ENV JAVA_VERSION ','').replace('+','-')  
  def withoutJdkPrefix = javaVersion.replaceAll("^jdk-", "").replaceAll("^jdk", "")
  if (withoutJdkPrefix.contains("_")) {
    return withoutJdkPrefix.split('_')[0]
  }
  return withoutJdkPrefix
}

def getTags(jdkShortVersion, jdkLongVersion, type, jdkType) {
  def tags = []
  def org = "${isTrusted()?'jenkins':'jenkins4eval'}"  
  if (env.BRANCH_NAME == 'master') {
    tags << "${org}/openjdk:${jdkShortVersion}-${type}-${jdkType}-windowsservercore-ltsc2019"  
    tags << "${org}/openjdk:${jdkLongVersion}-${type}-${jdkType}-windowsservercore-ltsc2019"
    if (jdkShortVersion == '15') {
      tags << "${org}/openjdk:${type}-${jdkType}-windowsservercore-ltsc2019"
      if (jdkType == 'hotspot') {
        tags << "${org}/openjdk:${type}-windowsservercore-ltsc2019"
        if (type == 'jdk') {
          tags << "${org}/openjdk:windowsservercore-ltsc2019"
        }
      }  
    }
    if (type == 'jdk') {
      tags << "${org}/openjdk:${jdkShortVersion}-${jdkType}-windowsservercore-ltsc2019"
      tags << "${org}/openjdk:${jdkLongVersion}-${jdkType}-windowsservercore-ltsc2019"
    }
  } else {
    tags << "${org}/openjdk:${jdkShortVersion}-${type}-${jdkType}-SNAPSHOT"
  }
  return tags
}
