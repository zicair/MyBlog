---
title: Halo开源项目学习-1
tags:
  - Java
  - Halo
  - SpringBoot
categories:
  - SpringBoot
date: 2020-08-31 23:31:07
---

Halo是使用Java语言开发的一个博客系统，基于SpringBoot框架。这里下载使用的是`halo-1.0.0-beta.7`的版本。

<!--more-->

# 1. 查看配置

**build.gradle配置**

主要添加使用到的一些工具的依赖

```java
plugins {
    id 'org.springframework.boot' version '2.1.3.RELEASE'
//    id "io.freefair.lombok" version "3.1.4"
    id 'java'
}

apply plugin: 'io.spring.dependency-management'

group = 'run.halo.app'
archivesBaseName = 'halo'
version = '1.0.0-beta.7'
sourceCompatibility = '1.8'
description = 'Halo, personal blog system developed in Java.'

repositories {
    maven {
        url 'https://maven.aliyun.com/nexus/content/groups/public'
    }
    mavenCentral()
    jcenter()
}

configurations {
    implementation {
        # 移除tomcat的依赖使用undertow
        exclude module: 'spring-boot-starter-tomcat'
    }

    developmentOnly
    runtimeClasspath {
        extendsFrom developmentOnly
    }
}

bootJar {
    manifest {
        attributes('Implementation-Title': 'Halo Application',
                'Implementation-Version': version)
    }
}

dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-actuator'
    implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
    implementation 'org.springframework.boot:spring-boot-starter-web'
    implementation 'org.springframework.boot:spring-boot-starter-undertow'
    implementation 'org.springframework.boot:spring-boot-starter-freemarker'

    implementation 'io.github.biezhi:oh-my-email:0.0.4'
    implementation 'cn.hutool:hutool-core:4.5.0'
    implementation 'cn.hutool:hutool-crypto:4.5.0'
    implementation 'cn.hutool:hutool-extra:4.5.0'
    implementation 'com.upyun:java-sdk:4.0.1'
    implementation 'com.qiniu:qiniu-java-sdk:7.2.18'
    implementation 'com.aliyun.oss:aliyun-sdk-oss:3.4.2'
    implementation 'net.coobird:thumbnailator:0.4.8'
    implementation 'com.atlassian.commonmark:commonmark:0.12.1'
    implementation 'com.atlassian.commonmark:commonmark-ext-gfm-tables:0.12.1'
    implementation 'com.atlassian.commonmark:commonmark-ext-yaml-front-matter:0.12.1'
    implementation 'io.springfox:springfox-swagger2:2.9.2'
    implementation 'io.springfox:springfox-swagger-ui:2.9.2'
    implementation 'org.apache.commons:commons-lang3:3.8.1'
    implementation 'org.apache.httpcomponents:httpclient:4.5.7'
    implementation 'com.fasterxml.jackson.dataformat:jackson-dataformat-yaml:2.9.2'
    implementation 'org.eclipse.jgit:org.eclipse.jgit:5.3.0.201903130848-r'


    compileOnly "org.projectlombok:lombok"
    annotationProcessor "org.projectlombok:lombok"

    testCompileOnly "org.projectlombok:lombok"
    testAnnotationProcessor "org.projectlombok:lombok"
    runtimeOnly 'com.h2database:h2'
    runtimeOnly 'mysql:mysql-connector-java'

    testImplementation 'org.springframework.boot:spring-boot-starter-test'

    developmentOnly 'org.springframework.boot:spring-boot-devtools'
}
```

**resource目录下的application.yaml配置**

主要设置服务器端口等信息，SpringBoot的数据源、数据库等信息

```yaml
server:
  port: 8090
  use-forward-headers: true
  undertow: # 使用Undertow代替tomcat
    io-threads: 2
    worker-threads: 36
    buffer-size: 1024
    directBuffers: true
  servlet:
    session:
      timeout: 86400s
  compression:
    enabled: true
    mime-types: application/javascript,text/css,application/json,application/xml,text/html,text/xml,text/plain
spring:
  devtools:
    add-properties: false
  output:
    ansi:
      enabled: always
  datasource:
    type: com.zaxxer.hikari.HikariDataSource

    # H2database 配置
    driver-class-name: org.h2.Driver
    url: jdbc:h2:file:~/.halo/db/halo
    username: admin
    password: 123456

    # MySql 配置
  #    driver-class-name: com.mysql.cj.jdbc.Driver
  #    url: jdbc:mysql://127.0.0.1:3306/halodb?characterEncoding=utf8&useSSL=false&serverTimezone=Asia/Shanghai
  #    username: root
  #    password: 123456

  h2:
    console:
      settings:
        web-allow-others: false
      path: /h2-console
      enabled: false
  jpa:
    hibernate:
      ddl-auto: update
    show-sql: false
    open-in-view: false
  servlet:
    multipart:
      max-file-size: 10240MB
      max-request-size: 10240MB
logging:
  level:
    run.halo.app: INFO
  file: ./logs/log.log
```

# 2. StartedListener

添加了一个`StartedListener` implements ApplicationListener<ApplicationStartedEvent>

监听程序的启动，并执行自定义的方法；

```java
package run.halo.app.listener;

@Slf4j
@Configuration
@Order(Ordered.HIGHEST_PRECEDENCE)
public class StartedListener implements ApplicationListener<ApplicationStartedEvent> {

    @Autowired
    private HaloProperties haloProperties;

    @Autowired
    private OptionService optionService;

    @Autowired
    private ThemeService themeService;

    @Autowired
    private UserService userService;

    @Override
    public void onApplicationEvent(ApplicationStartedEvent event) {
        // 主要执行两个方法，控制台打印端口信息和初始化主题
        this.printStartInfo();
        this.initThemes();
    }

    private void printStartInfo() {
        String blogUrl = optionService.getBlogBaseUrl();

        log.info("Halo started at         {}", blogUrl);
        log.info("Halo admin started at   {}/admin", blogUrl);
        if (!haloProperties.isDocDisabled()) {
            log.debug("Halo doc was enable at  {}/swagger-ui.html", blogUrl);
        }
    }

    /**
     * Init internal themes
     */
    private void initThemes() {
        // Whether the blog has initialized
        Boolean isInstalled = optionService.getByPropertyOrDefault(PrimaryProperties.IS_INSTALLED, Boolean.class, false);

        if (haloProperties.isProductionEnv() && isInstalled) {
            // Skip
            return;
        }

        try {
            String themeClassPath = ResourceUtils.CLASSPATH_URL_PREFIX + ThemeService.THEME_FOLDER;

            URI themeUri = ResourceUtils.getURL(themeClassPath).toURI();

            log.debug("Theme uri: [{}]", themeUri);

            Path source;

            if (themeUri.getScheme().equalsIgnoreCase("jar")) {
                // Create new file system for jar
                FileSystem fileSystem = FileSystems.newFileSystem(themeUri, Collections.emptyMap());
                source = fileSystem.getPath("/BOOT-INF/classes/" + ThemeService.THEME_FOLDER);
            } else {
                source = Paths.get(themeUri);
            }

            // Create theme folder
            Path themePath = themeService.getBasePath();

            if (!haloProperties.isProductionEnv() || Files.notExists(themePath)) {
                FileUtils.copyFolder(source, themePath);
                log.info("Copied theme folder from [{}] to [{}]", source, themePath);
            } else {
                log.info("Skipped copying theme folder due to existence of theme folder");
            }
        } catch (Exception e) {
            throw new RuntimeException("Initialize internal theme to user path error", e);
        }
    }

}
```

## printStartInfo()

```java
private void printStartInfo() {
        String blogUrl = optionService.getBlogBaseUrl();

        log.info("Halo started at         {}", blogUrl);
        log.info("Halo admin started at   {}/admin", blogUrl);
        if (!haloProperties.isDocDisabled()) {
            log.debug("Halo doc was enable at  {}/swagger-ui.html", blogUrl);
        }
    }
```

通过调用`optionService`的方法获取blogUrl：optionService.getBlogBaseUrl()

在`optionServiceImpl`实现类中实现如下：

```java
@Override
    public String getBlogBaseUrl() {
        // Get server port
        String serverPort = applicationContext.getEnvironment().getProperty("server.port", "8080");

        String blogUrl = getByProperty(BlogProperties.BLOG_URL).orElse("").toString();

        if (StrUtil.isNotBlank(blogUrl)) {
            blogUrl = StrUtil.removeSuffix(blogUrl, "/");
        } else {
            blogUrl = String.format("http://%s:%s", HaloUtils.getMachineIP(), serverPort);
        }

        return blogUrl;
    }
```

`getByProperty(BlogProperties.BLOG_URL)`最终调用`optionServiceImpl`的getByKey()方法：

```java
@Override
    public Optional<Object> getByKey(String key) {
        Assert.hasText(key, "Option key must not be blank");

        return Optional.ofNullable(listOptions().get(key));
    }
```

通过listOptions()列出所有的键值对，再通过key获取对应的值。

需要注意的是，在实现类`optionServiceImpl`中构造器会初始化

```java
private final Map<String, PropertyEnum> propertyEnumMap;
```

```java
public OptionServiceImpl(OptionRepository optionRepository,
                             ApplicationContext applicationContext,
                             StringCacheStore cacheStore,
                             ApplicationEventPublisher eventPublisher) {
        super(optionRepository);
        this.optionRepository = optionRepository;
        this.applicationContext = applicationContext;
        this.cacheStore = cacheStore;
        this.eventPublisher = eventPublisher;
        // Collections.unmodifiableMap()返回一个只读的map，
        // PropertyEnum.getValuePropertyEnumMap()定义的方法，用来遍历所有的enum元素，将每个ENUM.value做键，ENUM对象做值；
        propertyEnumMap = Collections.unmodifiableMap(PropertyEnum.getValuePropertyEnumMap());
    }
```

`listOptions()`实现：

```java
@Override
    @SuppressWarnings("unchecked")
    public Map<String, Object> listOptions() {
        // Get options from cache
        // cacheStore.getAny(OPTIONS_KEY, Map.class).orElseGet(()->...
        // 1. getAny先从缓存中取出所有数据，2. 不存在时从数据库中读取所有数据；
        return cacheStore.getAny(OPTIONS_KEY, Map.class).orElseGet(() -> {
            // 以下全部是第2.种数据库中读取的情况：
            List<Option> options = listAll();

            Set<String> keys = ServiceUtils.fetchProperty(options, Option::getKey);
            // 1). 将数据库中查到的所有数据转成map
            Map<String, Object> userDefinedOptionMap = ServiceUtils.convertToMap(options, Option::getKey, option -> {
                String key = option.getKey();

                PropertyEnum propertyEnum = propertyEnumMap.get(key);

                if (propertyEnum == null) {
                    return option.getValue();
                }

                return PropertyEnum.convertTo(option.getValue(), propertyEnum);
            });
            // 2). 定义返回的result，默认放入上一步从数据库中找出并转成map的数据
            Map<String, Object> result = new HashMap<>(userDefinedOptionMap);

            // 3). 对于数据库中没有包含的键值时，以默认值的方式存入result；
            propertyEnumMap.keySet()
                    .stream()
                    .filter(key -> !keys.contains(key))
                    .forEach(key -> {
                        PropertyEnum propertyEnum = propertyEnumMap.get(key);
                        // 筛选，排除默认值为空的键值对
                        if (StringUtils.isBlank(propertyEnum.defaultValue())) {
                            return;
                        }

                        result.put(key, PropertyEnum.convertTo(propertyEnum.defaultValue(), propertyEnum));
                    });

            // 4). 将result中的结果加入缓存
            cacheStore.putAny(OPTIONS_KEY, result);

            return result;
        });
    }
```

## initThemes()

```java
private void initThemes() {
        // Whether the blog has initialized
        // 同样从listOptions中获取相应的键值
        Boolean isInstalled = optionService.getByPropertyOrDefault(PrimaryProperties.IS_INSTALLED, Boolean.class, false);

        if (haloProperties.isProductionEnv() && isInstalled) {
            // 如果已经安装直接返回
            return;
        }

        try {
            String themeClassPath = ResourceUtils.CLASSPATH_URL_PREFIX + ThemeService.THEME_FOLDER;
            // themeClassPath: classpath:templates/themes
            URI themeUri = ResourceUtils.getURL(themeClassPath).toURI();
            // file:/E:/JAVAcode/halo-1.0.0 beta.7/out/production/resources/templates/themes
            log.debug("Theme uri: [{}]", themeUri);

            Path source;
            // "file".equalsIgnoreCase...
            if (themeUri.getScheme().equalsIgnoreCase("jar")) {
                // Create new file system for jar
                FileSystem fileSystem = FileSystems.newFileSystem(themeUri, Collections.emptyMap());
                source = fileSystem.getPath("/BOOT-INF/classes/" + ThemeService.THEME_FOLDER);
            } else { // E:\JAVAcode\halo-1.0.0-beta.7\out\production\resources\templates\themes
                source = Paths.get(themeUri);
            }

            // Create theme folder
            // C:\Users\zzt\.halo\templates\themes
            Path themePath = themeService.getBasePath();

            if (!haloProperties.isProductionEnv() || Files.notExists(themePath)) {
                FileUtils.copyFolder(source, themePath);
                log.info("Copied theme folder from [{}] to [{}]", source, themePath);
            } else {
                log.info("Skipped copying theme folder due to existence of theme folder");
            }
        } catch (Exception e) {
            throw new RuntimeException("Initialize internal theme to user path error", e);
        }
    }
```

themeService.getBasePath()方法中返回`ThemeServiceImpl`中的themeWorkDir

而`ThemeServiceImpl`实现类中的构造器初始化了themeWorkDir路径：C:\Users\zzt\.halo\templates\themes

```java
themeWorkDir = Paths.get(haloProperties.getWorkDir(), THEME_FOLDER);
```