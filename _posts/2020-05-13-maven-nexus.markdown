---
layout: post
title:  "Gradle配置上传到Nexus私服2020教程"
date:   2020-05-13 23:13:01 +0800
categories: jekyll update
---
Gradle配置上传到Nexus私服2020教程
===

## 2020配置Ijkplayer上传到私有Nexus服务器。 

#### 1. 在根目录build配置增加一下maven和bintray
```groovy
  dependencies {
        classpath 'com.android.tools.build:gradle:3.4.2'
        #需要上传的仓库信息. 
        classpath 'com.github.dcendents:android-maven-gradle-plugin:1.4.1'
        classpath 'com.jfrog.bintray.gradle:gradle-bintray-plugin:1.7'
    }
```

### 2. 编写账户和独立信息放到根目录的gradle.properties
```groovy
# nexus对应的Maven配置.
RELEASE_REPOSITORY_URL=http://172.16.22.18:8081/repository/android/
# 这里我的私有仓库只有release仓库，没有建snapshot和release的host仓库,有兴趣可以自行添加. 
SNAPSHOT_REPOSITORY_URL=http://172.16.22.18:8081/repository/android/
NEXUS_USERNAME=android
NEXUS_PASSWORD=android

```

### 3. 在需要的module下的`gradle.properties`中添加lib的相关信息
```groovy
POM_NAME=ijkplayer-java
POM_ARTIFACT_ID=ijkplayer-java
POM_PACKAGING=aar
```

### 4. 在根目录新建tools/gradle-mvn-push.gradle
```
- tools
  - gradle-mvn-push.gradle
  - gradle-bintray-upload.gradle
  - gradle-on-demand.gradle
```


- gradle-on-demand.gradle

```groovy

/*
 * Copyright 2016 Zhang Rui <bbcallen@gmail.com>
 *
 * Licensed under the Apache License, Version 2.0 (the "License");
 * you may not use this file except in compliance with the License.
 * You may obtain a copy of the License at
 *
 *     http://www.apache.org/licenses/LICENSE-2.0
 *
 * Unless required by applicable law or agreed to in writing, software
 * distributed under the License is distributed on an "AS IS" BASIS,
 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 * See the License for the specific language governing permissions and
 * limitations under the License.
 */

gradle.startParameter.taskNames.each { task ->
    def taskName = task.split(":").last()
    switch (taskName) {
        case "uploadArchives":
            apply from: new File(rootProject.projectDir, 'tools/gradle-mvn-push.gradle')
            break;
        case "bintrayUpload":
            apply from: new File(rootProject.projectDir, 'tools/gradle-bintray-upload.gradle')
            break;
        default:
            // do nothing
            break;
    }
}

```

- gradle-mvn-push.gradle

```groovy
/*
 * Copyright 2013 Chris Banes
 *
 * Licensed under the Apache License, Version 2.0 (the "License");
 * you may not use this file except in compliance with the License.
 * You may obtain a copy of the License at
 *
 *     http://www.apache.org/licenses/LICENSE-2.0
 *
 * Unless required by applicable law or agreed to in writing, software
 * distributed under the License is distributed on an "AS IS" BASIS,
 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 * See the License for the specific language governing permissions and
 * limitations under the License.
 */

apply plugin: 'maven'
apply plugin: 'signing'

def isReleaseBuild() {
    return VERSION_NAME.contains("SNAPSHOT") == false
}

def getReleaseRepositoryUrl() {
    return hasProperty('RELEASE_REPOSITORY_URL') ? RELEASE_REPOSITORY_URL
            : "https://oss.sonatype.org/service/local/staging/deploy/maven2/"
}

def getSnapshotRepositoryUrl() {
    return hasProperty('SNAPSHOT_REPOSITORY_URL') ? SNAPSHOT_REPOSITORY_URL
            : "https://oss.sonatype.org/content/repositories/snapshots/"
}

def getRepositoryUsername() {
    return hasProperty('NEXUS_USERNAME') ? NEXUS_USERNAME : ""
}

def getRepositoryPassword() {
    return hasProperty('NEXUS_PASSWORD') ? NEXUS_PASSWORD : ""
}

afterEvaluate { project ->
    uploadArchives {
        repositories {
            mavenDeployer {
                beforeDeployment { MavenDeployment deployment -> signing.signPom(deployment) }

                pom.groupId = GROUP
                pom.artifactId = POM_ARTIFACT_ID
                pom.version = VERSION_NAME

                repository(url: getReleaseRepositoryUrl()) {
                    authentication(userName: getRepositoryUsername(), password: getRepositoryPassword())
                }
                snapshotRepository(url: getSnapshotRepositoryUrl()) {
                    authentication(userName: getRepositoryUsername(), password: getRepositoryPassword())
                }

                pom.project {
                    name POM_NAME
                    packaging POM_PACKAGING
                    description POM_DESCRIPTION
                    url POM_URL

                    licenses {
                        license {
                            name POM_LICENSE_NAME
                            url POM_LICENSE_URL
                            distribution POM_LICENSE_DIST
                        }
                    }

                    scm {
                        url POM_SCM_URL
                        connection POM_SCM_CONNECTION
                        developerConnection POM_SCM_DEV_CONNECTION
                    }

                    developers {
                        developer {
                            id POM_DEVELOPER_ID
                            name POM_DEVELOPER_NAME
                            email POM_DEVELOPER_EMAIL
                        }
                    }
                }
            }
        }
    }

    android.libraryVariants.all { variant ->
        if(variant.buildType.name.equals("release")) {
            def jarTask = project.tasks.create(name: "jar${variant.name.capitalize()}", type: Jar) {
                from variant.javaCompile.destinationDir
                exclude "**/R.class"
                exclude "**/R\$**.class"
                exclude "**/BuildConfig.class"
            }
            jarTask.dependsOn variant.javaCompile
        }
    }

    signing {
        required { isReleaseBuild() && gradle.taskGraph.hasTask("uploadArchives") }
        sign configurations.archives
    }

    task androidJavadocs(type: Javadoc) {
        failOnError false
        source = android.sourceSets.main.java.srcDirs
        classpath += project.files(android.getBootClasspath().join(File.pathSeparator))
    }

    task androidJavadocsJar(type: Jar, dependsOn: androidJavadocs) {
        classifier = 'javadoc'
        from androidJavadocs.destinationDir
    }

    task androidSourcesJar(type: Jar) {
        classifier = 'sources'
        from android.sourceSets.main.java.sourceFiles
    }

    artifacts {
        archives androidSourcesJar
        archives androidJavadocsJar
        archives jarRelease
    }
}

```

- gradle-bintray-upload.gradle

```groovy
/*
 * Copyright 2015 Zhang Rui <bbcallen@gmail.com>
 *
 * Licensed under the Apache License, Version 2.0 (the "License");
 * you may not use this file except in compliance with the License.
 * You may obtain a copy of the License at
 *
 *     http://www.apache.org/licenses/LICENSE-2.0
 *
 * Unless required by applicable law or agreed to in writing, software
 * distributed under the License is distributed on an "AS IS" BASIS,
 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 * See the License for the specific language governing permissions and
 * limitations under the License.
 */

apply plugin: 'com.github.dcendents.android-maven'
apply plugin: 'com.jfrog.bintray'

group = GROUP
version = VERSION_NAME

bintray {
    user = project.hasProperty('BINTRAY_USER') ? BINTRAY_USER : System.getenv('BINTRAY_USER')
    key = project.hasProperty('BINTRAY_APIKEY') ? BINTRAY_APIKEY : System.getenv('BINTRAY_APIKEY')

    configurations = ['archives']

    dryRun = false
    publish = true

    pkg {
        repo = 'maven'
        name = POM_NAME
        userOrg = POM_USER_ORG
        desc = POM_DESCRIPTION
        websiteUrl = POM_URL
        vcsUrl = POM_SCM_URL
        licenses = [POM_LICENSE_NAME]
        labels = ['FFmpeg', 'Android', 'player']
        publicDownloadNumbers = true
        version {
            name = VERSION_NAME
            gpg {
                sign = true
                passphrase = project.hasProperty('GPG_PASSWORD') ? GPG_PASSWORD : System.getenv('GPG_PASSWORD')
            }
        }
    }
}

install {
    repositories.mavenInstaller {
        pom.project {
            name POM_NAME
            packaging POM_PACKAGING
            description POM_DESCRIPTION
            url POM_URL

            licenses {
                license {
                    name POM_LICENSE_NAME
                    url POM_LICENSE_URL
                    distribution POM_LICENSE_DIST
                }
            }

            scm {
                url POM_SCM_URL
                connection POM_SCM_CONNECTION
                developerConnection POM_SCM_DEV_CONNECTION
            }

            developers {
                developer {
                    id POM_DEVELOPER_ID
                    name POM_DEVELOPER_NAME
                    email POM_DEVELOPER_EMAIL
                }
            }
        }
    }
}

```

### 5. 在各自module的build.gradle里面新建任务和配置maven信息引入
```groovy
//新建一个任务，这里只需要定义名字，下面的配置文件中会自动获取相关配置信息. 
uploadArchives {

}
apply from: new File(rootProject.projectDir, "tools/gradle-on-demand.gradle");

```

### 6. 在右侧的Gradle脚本里面找到`upload/uploadArchives`双击上传

### 7. 查看nexus的库是否上传成功了. 
![861d7570.png](/assets/blog_res/861d7570.png)


### 8 . 多模块互相依赖注意事项
- 先分别上传其他单独module ex:ijk-java/ijk-armv7a
- 最后上传ijk-demo/项目中以来改为地址依赖(方便pom记录以来信息)


## 最后的最后还是翻车了，so没有上传上去只有java代码上去了.aar上传搞定. 
- 其实是刷新问题，等一会刷新下页面就能看到aar的信息了，直接引用@aar即可

>上绝招了：直接到module/build/output/aar/xxx.aar拷贝上传到nexus
Classifier不用填其他照着配置填好,点击上传即可

![d8467fa0.png](/assets/blog_res/d8467fa0.png)


- 引用的时候在包名后面添加`xxx@aar`
```
implementation 'tv.danmaku.ijk.media:ijkplayer-arm64:0.8.8@aar'
```


总结：没有lib/xx.so的module使用gradle一键上传jar包即可，如果是
有so库要上传的需要手动上传module.aar即可。

### 等待探求问题
- 有一个奇怪的现象，我上传ijkplayerview的时候用的也是脚本命令，却自动给我上传了aar文件.
初步猜测是因为womodulize里面配置了service和一些string资源. 


- 友联 & 相关开源项目源码出处. 

- [GitHub - jdpxiaoming/PPlayer: ffmpeg 4.0.2静态库从0开始一个播放器的搭建，支持rtmp、rtsp、hls、本地MP4文件播放，音视频同步，直播推流](https://github.com/jdpxiaoming/PPlayer)

- [GitHub - jdpxiaoming/ijkrtspdemo: ijkplayer open the rtsp & h265 surpport android demo .](https://github.com/jdpxiaoming/ijkrtspdemo)

- [GitHub - jdpxiaoming/FFmpegTools: use ffmpege as the binary file use commands params to run main funcition , picture and videos acitons as you licke.](https://github.com/jdpxiaoming/FFmpegTools/)

