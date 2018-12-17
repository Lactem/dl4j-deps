### Purpose
Creates a single JAR of dependencies for DeepLearning4Java (DL4J).
Can be used as a single dependency that unifies ~100 DL4J-related libraries.

### Benefits for Projects Using DeepLearning4Java
* Drastically reduces the size of Gradle shadow JARs (or Maven uber/fat/shaded JARs).
* Reduces compile time by up to 3 minutes.
* Eliminates the need to search for the long lists of transitive dependencies of various DL4J libraries.

### Setup / Installation
You first need to add dl4j-deps as a dependency to your project. I do this through an Artifactory server
(clone the project and run `./gradlew artifactoryPublish` using settings from ~/.gradle/gradle.properties),
but you can also add dl4j-deps as a
[local dependency](https://docs.gradle.org/current/userguide/repository_types.html#sec:flat_dir_resolver).  

Next, add the following to your project's build.gradle:
```
jar {
    manifest {
        attributes 'Class-Path': 'lib/dl4j-deps-1.0-SNAPSHOT.jar'
    }
}

configurations {
    runtime.exclude module: 'dl4j-deps'  
}
```
This adds the dl4j-deps JAR to your project's classpath and excludes dl4j-deps from your shadow JAR.
You'll have to make a folder called 'lib' in the same directory as your JAR and add dl4j-deps to it. Assuming
you've cloned this project and you're currently in the directory where this README is located, your commands
would look something like:
```
$ ./gradlew shadowJar
$ mkdir /path/to/your_project/lib
$ cp build/libs/dl4j-deps-1.0-SNAPSHOT.jar /path/to/your_project/lib/dl4j-deps-1.0-SNAPSHOT.jar
```
Note that the first command, which builds the dl4j-deps JAR, will take a couple minutes. You should be good to go
at this point. If you changed dependencies in dl4j-deps and need to update the project that relies on dl4j-deps, go
to your project and run `./gradlew build --refresh-dependencies`.

### Caveats
* Some transitive dependencies can't be accessed in projects that use dl4j-deps unless they're explicity listed.
* Initial overhead is large since building the dl4j-deps shadow JAR and publishing it to Artifactory or Nexus can take
~15 minutes (this is the trade-off for reducing time in the long run, since dl4j-deps is only built once
while your other projects are built many times).