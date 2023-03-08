# How to use Gradle

```groovy
configurations {
    compileOnly {
        extendsFrom annotationProcessor
    }
}
```

`compileOnly` means that the library specified is needed to compile the project but the libraries code DOES NOT need to be included in the final build when the JAR file is generated.

`annotationProcessors` specifies that the dependency is needed during compile time to generate additional files for our project to run correctly.

## References

Annotation Processors: https://tomgregory.com/annotation-processors-in-gradle-with-the-annotationprocessor-dependency-configuration/

CompileOnly:

https://blog.gradle.org/introducing-compile-only-dependencies

https://discuss.gradle.org/t/gradle-annotationprocessor-works-but-implementation-dont-for-annotation-processors/43284
