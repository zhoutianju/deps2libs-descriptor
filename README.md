# 依赖分开打包 `maven-assembly-plugin` 描述器

## 用途

将依赖和自身 `jar` 包分开打包，打到 `tar.gz` 包，适用于依赖比较多的工程，更新自身 `jar` 包时，可以不用同时更新依赖 `jar` 包

## 安装

```bash
mvn install
```

## 使用方法

1. `pom.xml`

    ```xml
    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-jar-plugin</artifactId>
                <version>3.1.1</version>
                <configuration>
                    <archive>
                        <manifest>
                            <addClasspath>true</addClasspath>
                            <classpathPrefix>lib/</classpathPrefix>
                            <mainClass>com.github.zhoutianju.PackageDemo</mainClass>
                        </manifest>
                    </archive>
                </configuration>
            </plugin>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-install-plugin</artifactId>
                <version>2.5.2</version>
                <configuration>
                    <skip>true</skip>
                </configuration>
            </plugin>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-deploy-plugin</artifactId>
                <version>2.8.2</version>
                <configuration>
                    <skip>true</skip>
                </configuration>
            </plugin>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-assembly-plugin</artifactId>
                <dependencies>
                    <dependency>
                        <groupId>com.github.zhoutianju</groupId>
                        <artifactId>deps2libs-descriptor</artifactId>
                        <version>1.0.0-SNAPSHOT</version>
                    </dependency>
                </dependencies>
                <configuration>
                    <descriptorRefs>
                        <descriptorRef>deps2libs-assembly</descriptorRef>
                    </descriptorRefs>
                </configuration>
                <executions>
                    <execution>
                        <id>make-assembly</id>
                        <phase>deploy</phase>
                        <goals>
                            <goal>single</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>
    ```
1. 首次打包（或变更依赖后）部署

    ```bash
    # 首次（或变更依赖后）打包
    mvn deploy
    # 首次（或变更依赖后）部署（运行环境）复制 xxx.tar.gz 到执行目录
    tar -xzvf xxx.tar.gz
    java -jar xxx.jar
    ```
1. 更新代码后打包部署

    ```bash
    # 更新代码后打包
    mvn package
    # 更新代码后部署（运行环境）复制 xxx.jar 到执行目录
    java -jar xxx.jar
    ```

## 解释

1. 建议在 `maven-jar-plugin` 插件中配置 `classpathPrefix` 和 `mainClass` ，否则需要在运行参数中同时指定 `mainClass` 和 `classpath` 
（这种方式不会读取 `jar` 包中的 `META-INF\MANIFEST.MF`），很麻烦，比如：

    ```bash
    java -jar xxx.jar xxx.MainClass -cp .:lib/a.jar:lib/b.jar
    ```
1. 关于在 `maven-install-plugin` 插件中配置`<skip>true</skip>`，是因为 `maven` 在 `super pom` 中默认将 `maven-install-plugin` 插件绑定到 `install` 阶段，
而一般使用这种方式打包的程序是直接用来运行的，所以 `install` 也就没有意义了，关闭可加快打包速度，如果有特殊需求（如其他包需要依赖，虽然这种做法很蠢），也可以打开
1. 关于在 `maven-deploy-plugin` 插件中配置 `<skip>true</skip>`，是因为 `maven` 在 `super pom` 中默认将 `maven-deploy-plugin` 插件绑定到 `deploy` 阶段，
而一般使用这种方式打包的程序是直接用来运行的，所以 `deploy` 没意义，现在将 `maven-assembly-plugin` 绑定在了 `deploy` 阶段，如果有其他需求可一起绑定到 `deploy` 阶段即可（注意执行顺序）