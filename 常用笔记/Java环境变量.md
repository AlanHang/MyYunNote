JAVA_HOME  jdk安装路径    

Path 添加 %JAVA_HOME%\bin;%JAVA_HOME%\jre\bin;

ClassPath  .;%JAVA_HOME%\lib;%JAVA_HOME%lib\tools.jar  

eclipse 版本 eclipse-jee-2019-06-R-win32-x86_64


```
Java 配置（最好配置到当前用户环境变量中）
JAVA_HOME  jdk安装路径   
PATH 添加 %JAVA_HOME%\bin;//当前用户不存在Path变量时，可以自定义一个，然后%PATH%
验证： java -version
       echo "%JAVA_HOME%"

MAVEN配置（最好配置到当前用户环境变量中）
M2_HOME
PATH 添加 %M2_HOME%\bin;
验证： mvn -version
       echo "%M2_HOME%"
```

