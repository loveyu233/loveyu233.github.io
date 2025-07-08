---
title: "SpringBoot启动结束钩子"
date: 2019-08-15T13:48:12+08:00
draft: true
---

befor/end

```java
@Component
@Slf4j
public class Action {
    @PostConstruct
    public void con() {
        log.info("@PostConstruct");
    }

    @PreDestroy
    public void pre() {
        log.info("@PreDestroy");
    }
}

@Component
@Slf4j
class con implements ApplicationRunner {
    @Override
    public void run(ApplicationArguments args) throws Exception {
        log.info("implements ApplicationRunner");
    }
}

@Component
@Slf4j
class de implements CommandLineRunner {
    @Override
    public void run(String... args) throws Exception {
        log.info("@Override");
    }
}
```



在resource下创建banner.txt

```txt
${AnsiColor.BRIGHT_RED}
_____.___._____.___.________    _________
\__  |   |\__  |   |\______ \  /   _____/
 /   |   | /   |   | |    |  \ \_____  \
 \____   | \____   | |    `   \/        \
 / ______| / ______|/_______  /_______  /
 \/        \/               \/        \/
 _______        __________
 \      \   ____\______   \__ __  ____
 /   |   \ /  _ \|    |  _/  |  \/ ___\
/    |    (  <_> )    |   \  |  / /_/  >
\____|__  /\____/|______  /____/\___  /
        \/              \/     /_____/
${AnsiColor.BRIGHT_YELLOW}
::: 版本信息: --------- SpringBoot ${spring-boot.formatted-version}
```

