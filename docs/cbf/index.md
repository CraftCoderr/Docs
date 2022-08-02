# Cosmic Binary Format

## Configuring Maven / Gradle

To use CBF, you first have to add the xenondevs maven repository to your build configuration.

=== "Maven"

    ```xml
    <repository>
        <id>xenondevs</id>
        <url>https://repo.xenondevs.xyz/releases</url>
    </repository>
    ```

=== "Gradle Groovy"

    ```groovy
    maven {
        url 'https://repo.xenondevs.xyz/releases'
    }
    ```

=== "Gradle Kotlin"

    ```kotlin
    maven {
        url = uri("https://repo.xenondevs.xyz/releases")
    }
    ```

Now you can add CBF to your build configuration:

=== "Maven"

    ```xml
    <dependency>
        <groupId>xyz.xenondevs.cbf</groupId>
        <artifactId>cosmic-binary-format</artifactId>
        <version>VERSION</version>
    </dependency>
    ```

=== "Gradle Groovy"

    ```groovy
    implementation "xyz.xenondevs.cbf:cosmic-binary-format:VERSION"
    ```

=== "Gradle Kotlin"

    ```kotlin
    implementation("xyz.xenondevs.cbf:cosmic-binary-format:VERSION")
    ```

To get started, you also need to install a CBF adapter. Currently, only the netty adapter is available.

=== "Maven"

    ```xml
    <dependency>
        <groupId>xyz.xenondevs.cbf</groupId>
        <artifactId>cosmic-binary-format-netty-adapter</artifactId>
        <version>VERSION</version>
    </dependency>
    ```

=== "Gradle Groovy"

    ```groovy
    implementation "xyz.xenondevs.cbf:cosmic-binary-format-netty-adapter:VERSION"
    ```

=== "Gradle Kotlin"

    ```kotlin
    implementation("xyz.xenondevs.cbf:cosmic-binary-format-netty-adapter:VERSION")
    ```