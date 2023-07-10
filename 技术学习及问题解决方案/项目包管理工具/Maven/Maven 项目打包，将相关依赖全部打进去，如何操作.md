## Maven 项目打包，将相关依赖全部打进去，如何操作

要将所有依赖项打包到 Maven 项目中，可以使用 Maven 的 `maven-assembly-plugin` 插件。这个插件可以帮助将所有依赖项打包到一个可执行的 JAR 文件中。

以下是一个简单的步骤，说明如何使用 `maven-assembly-plugin` 将所有依赖项打包到一个 JAR 文件中：

1. 首先，在你的 Maven 项目中，打开 `pom.xml` 文件。

2. 在 `<build>` 标签下，添加以下代码来配置 `maven-assembly-plugin` 插件：

   ```xml
   <plugins>
       <plugin>
           <artifactId>maven-assembly-plugin</artifactId>
           <version>3.1.1</version>
           <configuration>
               <archive>
                   <manifest>
                       <mainClass>com.example.MainClass</mainClass>
                   </manifest>
               </archive>
               <descriptorRefs>
                   <descriptorRef>jar-with-dependencies</descriptorRef>
               </descriptorRefs>
           </configuration>
           <executions>
               <execution>
                   <id>make-assembly</id>
                   <phase>package</phase>
                   <goals>
                       <goal>single</goal>
                   </goals>
               </execution>
           </executions>
       </plugin>
   </plugins>
   ```

   在这个配置中，我们指定了 `jar-with-dependencies` 描述符来打包所有依赖项。我们也指定了一个主类名，这个主类将在打包后的 JAR 文件中被执行。

3. 接下来，使用以下命令运行 Maven 项目的打包操作：

   ```
   mvn package
   ```

   这将使用 `maven-assembly-plugin` 插件来打包所有依赖项，并将它们全部打包到一个 JAR 文件中。你可以在 `target` 文件夹中找到这个 JAR 文件。

   ```
   target/your-project-name-jar-with-dependencies.jar
   ```

现在，你已经成功将所有依赖项打包到 Maven 项目中。

