---
title: Spring Boot 实现看门狗功能 (调用 Shell 脚本)
date: 2020-06-12 15:00:00
updated: 2020-06-12 15:00:00
categories: [IT]
tags: [Spring Boot, Shell, inotify]
---

需要实现看门狗功能，定时检测另外一个程序是否在运行，使用 crontab 仅可以实现检测程序是否正在运行，无法做到扩展，如：手动重启、程序升级(如果只需要实现自动升级功能可以使用 [inotify](https://blog.csdn.net/XlxfyzsFdblj/article/details/82110034))等功能；最后决定使用 Spring Boot 调用 Shell 脚本来实现

# 一、脚本

## 1.1 启动脚本

```
#!/bin/bash

ps -ef | grep "demo-app-0.0.1-SNAPSHOT.jar" | grep -v "grep"

if [ "$?" -eq 0 ]
then

# sleep
echo $(date "+%Y-%m-%d %H:%M:%S") "process already started!"

else

nohup java -jar -server /project/watchdog/demo-app-0.0.1-SNAPSHOT.jar &
echo $(date "+%Y-%m-%d %H:%M:%S") "process has been started!"

fi
```

## 1.2 重启脚本

```
#!/bin/bash

pid=`ps -ef | grep "demo-app-0.0.1-SNAPSHOT.jar" | grep -v "grep" | awk '{print $2}'`

for id in $pid
do
    kill -9 $id
    echo "killed $id"
done

nohup java -jar -server /project/watchdog/demo-app-0.0.1-SNAPSHOT.jar &
echo $(date "+%Y-%m-%d %H:%M:%S") "process has been restarted!"
```

# 二、功能实现

将脚本放置在程序的资源目录下，每次程序启动时将脚本读取到指定位置，然后再通过定时任务执行脚本

配置内容：

```
shell:
  directory: /project/watchdog
  startupFileName: startup.sh
  restartFileName: restart.sh
```

```
@Configuration
@ConfigurationProperties(prefix = "shell")
public class ShellProperties {

    private String directory;
    private String startupFileName;
    private String restartFileName;

    /* getter & setter */

    public String getFullName(String fileName) {
        return directory + File.separator + fileName;
    }
}
```

## 2.1 启动时将脚本读取到指定位置

```
@Component
public class InitRunner implements CommandLineRunner {

    @Autowired
    private ShellProperties shellProperties;

    @Autowired
    ResourceLoader resourceLoader;

    @Override
    public void run(String... args) throws Exception {
        generateFile(shellProperties.getStartupFileName());
        generateFile(shellProperties.getRestartFileName());
    }


    private void generateFile(String fileName) throws IOException {

        String fileFullName = shellProperties.getFullName(fileName);
        File file = new File(fileFullName);
        if(file.exists()) {
            return;
        }

        // 如果文件已存在，FileWriter 会先删除再新建
        FileWriter fileWriter = new FileWriter(fileFullName);

        Resource resource = resourceLoader.getResource("classpath:" + fileName);
        InputStream inputStream = resource.getInputStream();
        InputStreamReader inputStreamReader = new InputStreamReader(inputStream);
        BufferedReader bufferedReader = new BufferedReader(inputStreamReader);

        String data;
        while ((data = bufferedReader.readLine()) != null) {
            fileWriter.write(data + "\n");
        }

        bufferedReader.close();
        inputStreamReader.close();
        inputStream.close();

        fileWriter.close();

        // 设置权限，否则会报 Permission denied
        file.setReadable(true);
        file.setWritable(true);
        file.setExecutable(true);
    }
}
```

## 2.2 定时任务定时任务执行脚本

```
@Component
public class ShellTask {

    private static final Logger logger = LoggerFactory.getLogger(ShellTask.class);

    @Autowired
    private ShellProperties shellProperties;

    @Scheduled(cron = "0/10 * * * * ? ")
    public void start() throws IOException {
        executeShell(shellProperties.getStartupFileName());
    }

    private void executeShell(String fileName) throws IOException {

        String fileFullName = shellProperties.getFullName(fileName);
        File file = new File(fileFullName);
        if(!file.exists()) {
            logger.error("file {} not existed!", fileFullName);
            return;
        }

        ProcessBuilder processBuilder = new ProcessBuilder(fileFullName);
        processBuilder.directory(new File(shellProperties.getDirectory()));

        Process process = processBuilder.start();

//        String input;
//        BufferedReader stdInput = new BufferedReader(new InputStreamReader(process.getInputStream()));
//        BufferedReader stdError = new BufferedReader(new InputStreamReader(process.getErrorStream()));
//        while ((input = stdInput.readLine()) != null) {
//            logger.info(input);
//        }
//        while ((input = stdError.readLine()) != null) {
//            logger.error(input);
//        }

        int runningStatus = 0;
        try {
            runningStatus = process.waitFor();
        } catch (InterruptedException e) {
            logger.error("shell", e);
        }

        if(runningStatus != 0) {
            logger.error("failed.");
        }else {
            logger.info("success.");
        }
    }
}
```

## 2.3 扩展

1. 本例只实现了定时检测程序是否运行，如果没有运行则启动程序；如有需要可以添加接口，调用接口重启程序；或者添加定时任务定时检测程序是否有更新，如果有更新则下载新的 jar 包然后重启程序
1. 看门狗程序自己可以使用 crontab 定时检测是否正在运行，模仿上面的启动脚本编写看门狗的启动脚本，然后添加定时任务：

```
crontab -e
*/10 * * * * /project/watchdog/watchdog.sh
sudo systemctl reload crond.service
```


完整代码：[GitHub](https://github.com/VictorBu/code-snippet/tree/master/java/spring-boot-watchdog)

参考：[java去调用并执行shell脚本以及问题总结](https://blog.csdn.net/vcfriend/article/details/81226632)
