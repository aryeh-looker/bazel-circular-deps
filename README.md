# Working with Dependencies w/Circular Deps

maven when adding a dep with a circular dependency "just works":
```
aryehh@aryehh ~/Development/circular-deps/maven
 % mvn install
[INFO] Scanning for projects...
[INFO] 
[INFO] -------------------------< org.example:maven >--------------------------
[INFO] Building maven 1.0-SNAPSHOT
[INFO] --------------------------------[ jar ]---------------------------------
[INFO] 
[INFO] --- maven-resources-plugin:2.6:resources (default-resources) @ maven ---
[WARNING] Using platform encoding (UTF-8 actually) to copy filtered resources, i.e. build is platform dependent!
[INFO] Copying 0 resource
[INFO] 
[INFO] --- maven-compiler-plugin:3.1:compile (default-compile) @ maven ---
[INFO] Nothing to compile - all classes are up to date
[INFO] 
[INFO] --- maven-resources-plugin:2.6:testResources (default-testResources) @ maven ---
[WARNING] Using platform encoding (UTF-8 actually) to copy filtered resources, i.e. build is platform dependent!
[INFO] skip non existing resourceDirectory /usr/local/google/home/aryehh/Development/circular-deps/maven/src/test/resources
[INFO] 
[INFO] --- maven-compiler-plugin:3.1:testCompile (default-testCompile) @ maven ---
[INFO] Nothing to compile - all classes are up to date
[INFO] 
[INFO] --- maven-surefire-plugin:2.12.4:test (default-test) @ maven ---
[INFO] No tests to run.
[INFO] 
[INFO] --- maven-jar-plugin:2.4:jar (default-jar) @ maven ---
[INFO] 
[INFO] --- maven-install-plugin:2.4:install (default-install) @ maven ---
[INFO] Installing /usr/local/google/home/aryehh/Development/circular-deps/maven/target/maven-1.0-SNAPSHOT.jar to /usr/local/google/home/aryehh/.m2/repository/org/example/maven/1.0-SNAPSHOT/maven-1.0-SNAPSHOT.jar
[INFO] Installing /usr/local/google/home/aryehh/Development/circular-deps/maven/pom.xml to /usr/local/google/home/aryehh/.m2/repository/org/example/maven/1.0-SNAPSHOT/maven-1.0-SNAPSHOT.pom
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  0.958 s
[INFO] Finished at: 2022-09-23T22:33:47Z
[INFO] ------------------------------------------------------------------------
aryehh@aryehh ~/Development/circular-deps/maven
 % echo $?
0
```

bazel when adding a circular dep fails unless the circular dep is explicitly broken:
```
 % bazel build //...
ERROR: /usr/local/google/home/aryehh/.cache/bazel/_bazel_aryehh/7ac1967b990f2453d2320d3da05e6121/external/maven/BUILD:25:11: in jvm_import rule @maven//:org_antlr_stringtemplate: cycle in dependency graph:
    //:repro (6167a43bee81d7c4ceebdf8986e0027b728c609e76e2c99eb3032efb76afa9e5)
.-> @maven//:org_antlr_stringtemplate (6167a43bee81d7c4ceebdf8986e0027b728c609e76e2c99eb3032efb76afa9e5)
|   @maven//:org_antlr_antlr_runtime (6167a43bee81d7c4ceebdf8986e0027b728c609e76e2c99eb3032efb76afa9e5)
`-- @maven//:org_antlr_stringtemplate (6167a43bee81d7c4ceebdf8986e0027b728c609e76e2c99eb3032efb76afa9e5)
ERROR: Analysis of target '//:repro' failed; build aborted
INFO: Elapsed time: 5.429s
INFO: 0 processes.
FAILED: Build did NOT complete successfully (2 packages loaded, 5 targets configured)
```

## Repro: `mvn`
cd `mvn`; mvn install

## Repro: `bazel`
cd `bazel`; bazel build //...

## Workaround

```
         maven.artifact(
             "org.antlr",
             "antlr-runtime",
             "3.3",
             exclusions = [
                 "org.antlr:stringtemplate",
             ],
         ),
```

See `bazel/WORKSPACE` for more information
