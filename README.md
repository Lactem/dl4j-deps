### Purpose
Dl4j-deps creates a single JAR of dependencies used by DeepLearning4Java (DL4J). This JAR can be used as a single
dependency that unifies ~100 libraries, most of which are needed by programs that use DeepLearning4Java's API.  

Use Cases:
* Eliminate the need to include DL4J's heavy dependency set every time you compile your project by adding the dl4j-deps
JAR to your project's classpath. 
* Use directly or as a reference for creating a standalone DL4J project using Gradle. Dl4j-deps can be used as a
rough Gradle counterpart for the
[sample Maven standalone project](https://github.com/deeplearning4j/dl4j-examples/tree/master/standalone-sample-project).

### How to Use
To use dl4j-deps as-is, all you need to do is add the dependency and the repository that it's stored in.  
If your project is built with Gradle:
```Gradle
repositories {
    // TradeCatcher repo for dl4j-deps
    maven {
        url = 'http://stocks.maxilie.com:8081/artifactory/gradle-dev-local'
    }
}

dependencies {
    // DeepLearning4Java dependencies
    compile 'TradeCatcher:dl4j-deps:1.0-SNAPSHOT'
}
```
If your project is built with Maven:
```XML
<repositories>
    <repository>
        <id>TradeCatcher</id>
        <url>http://stocks.maxilie.com:8081/artifactory/gradle-dev-local</url>
    </repository>
</repositories>

<dependencies>
    <!-- DeepLearning4Java dependencies -->
    <dependency>
        <groupId>TradeCatcher</groupId>
        <artifactId>dl4j-deps</artifactId>
        <version>1.0-SNAPSHOT</version>
    </dependency>
</dependencies>
```

#### Using Provided Scope
The main use case for dl4j-deps is avoiding the need to include DL4J's large dependencies in your fat JAR. To do this
in Gradle, change the dependency scope from 'compile' to 'compileClasspath' (or add '<scope>provided</scope>' for maven).
Then add the following to your project's build.gradle:
```Gradle
jar {
    manifest {
        attributes 'Class-Path': 'lib/dl4j-deps-1.0-SNAPSHOT.jar'
    }
}

configurations {
    runtime.exclude module: 'dl4j-deps'  
}
```
This adds the dl4j-deps JAR to your project's classpath and excludes dl4j-deps from your shadow JAR (this second
part is only necessary if you use the shadow plugin). You'll have to make a folder called 'lib' in the same directory
as your JAR and add dl4j-deps to it. Assuming you've cloned this project and you're currently in the directory where
this README is located, your commands would look something like this:
```Shell
$ ./gradlew shadowJar
$ mkdir /path/to/your_project/lib
$ cp build/libs/dl4j-deps-1.0-SNAPSHOT.jar /path/to/your_project/lib/dl4j-deps-1.0-SNAPSHOT.jar
```
Note that the first command, which builds the dl4j-deps JAR, will take a couple minutes (and if you're using Windows
then the command would instead be `gradlew.bat shadowJar`). You should be good to go at this point!

#### Modifying dl4j-deps
If you want to fork dl4j-deps and modify it, you'll need somewhere to publish your changes. I do this through an
Artifactory server (`./gradlew artifactoryPublish` using settings from ~/.gradle/gradle.properties), but you can also
publish locally and then add dl4j-deps as a
[local dependency](https://docs.gradle.org/current/userguide/repository_types.html#sec:flat_dir_resolver). Remember
to run `./gradlew build --refresh-dependencies` in your project that uses dl4j-deps so that it will download your
changes. 

### Caveats
* I include some dependencies that aren't necessary for DL4J since I use this in another project. These extra
dependencies are small in comparison to DL4J, though. They include: time4j, iextrading4j, and MongoDB.
* If you're modifying dl4j-deps directly, initial overhead is large. Building the dl4j-deps shadow JAR and publishing
it to Artifactory or Nexus can take ~15 minutes (this is the trade-off for reducing time in the long run, since
dl4j-deps is only built once while your other projects are built many times).
* The ND4J backend is set to native-platform, which uses the CPU for deep learning. You'll have to make some adjustments
if you want to use the GPU (change native-platform to CUDA).