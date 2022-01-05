

代码无侵入，能生成多种形式

```xml
    <plugin>
        <groupId>com.github.shalousun</groupId>
        <artifactId>smart-doc-maven-plugin</artifactId>
        <version>2.2.1</version>
        <configuration>
            <!--指定生成文档使用的配置文件-->
            <configFile>./src/main/resources/smart-doc.json</configFile>
            <!--指定分析的依赖模块（避免分析所有依赖，导致生成文档变慢，循环依赖导致生成失败等问题）-->
            <includes>
                <!--格式为：groupId:artifactId;参考如下-->
                <include>com.alibaba:fastjson</include>
            </includes>
        </configuration>
        <executions>
            <execution>
                <!--不需要在编译项目时自动生成文档可注释phase-->
                <!-- <phase>compile</phase> -->
                <goals>
                    <goal>html</goal>
                </goals>
            </execution>
        </executions>
    </plugin>
```

```json
    {
        "serverUrl": "http://127.0.0.1:8888/serveName",
        "isStrict": false,
        "allInOne": true,
        "outPath": "src/main/resources/static/doc",
        "createDebugPage": true, //生成测试页面
        "projectName": "smart-doc"
    }
```

> 生成文档：  
>   maven -> Plugins -> smart-doc -> 各种格式
