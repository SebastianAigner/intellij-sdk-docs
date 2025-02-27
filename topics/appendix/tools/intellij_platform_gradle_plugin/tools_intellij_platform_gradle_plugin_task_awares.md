<!-- Copyright 2000-2024 JetBrains s.r.o. and contributors. Use of this source code is governed by the Apache 2.0 license. -->

# Task Awares

<link-summary>IntelliJ Platform Gradle Plugin task `*Aware` interfaces.</link-summary>

<include from="tools_intellij_platform_gradle_plugin.md" element-id="EAP_Status"/>
<include from="tools_intellij_platform_gradle_plugin.md" element-id="faq"/>

The Task Awares is a set of interfaces that can be applied to custom Gradle tasks and, when registered using the dedicated register method, inject new features or properties with predefined values.

IntelliJ Platform Gradle Plugin supports creating custom tasks which can use `*Aware` interfaces:

```kotlin
abstract class RetrievePluginNameTask : DefaultTask(), PluginAware

val retrievePluginName by tasks.registering(RetrievePluginNameTask::class) {
    val outputFile = layout.buildDirectory.file("pluginName.txt")

    doLast {
        outputFile.get().asFile.writeText(pluginXml.parse { name }.get())
    }
}
```

## `AutoReloadAware`
{#AutoReloadAware}

<tldr>

**Inherited by**: [`RunnableIdeAware`](#RunnableIdeAware)

**Sources**: [`AutoReloadAware`](%gh-ijpgp%/src/main/kotlin/org/jetbrains/intellij/platform/gradle/tasks/aware/AutoReloadAware.kt)

</tldr>

The interface provides the possibility to auto-reload plugin when run in the IDE.

### `autoReload`
{#AutoReloadAware-autoReload}

Enables auto-reload of dynamic plugins.
Dynamic plugin will be reloaded automatically when its content is modified.

This allows a much faster development cycle by avoiding a full restart of the development instance after code changes.

{style="narrow"}
Type
: `Property<Boolean>`

Default value
: [`intellijPlatform.autoReload`](tools_intellij_platform_gradle_plugin_extension.md#intellijPlatform-autoReload)


## `CoroutinesJavaAgentAware`
{#CoroutinesJavaAgentAware}

<tldr>

**Depends on**: [`initializeIntelliJPlatformPlugin`](tools_intellij_platform_gradle_plugin_tasks.md#initializeIntelliJPlatformPlugin)

**Inherited by**: [`RunnableIdeAware`](#RunnableIdeAware)

**Sources**: [`CoroutinesJavaAgentAware`](%gh-ijpgp%/src/main/kotlin/org/jetbrains/intellij/platform/gradle/tasks/aware/CoroutinesJavaAgentAware.kt)

</tldr>

The interface provides the path to the Java Agent file for the Coroutines library required to enable coroutines debugging.

### `coroutinesJavaAgentFile`
{#CoroutinesJavaAgentAware-coroutinesJavaAgentFile}

The path to the coroutines Java Agent file.

{style="narrow"}
Type
: `RegularFileProperty`

Default value
: [`initializeIntellijPlatformPlugin.coroutinesJavaAgent`](tools_intellij_platform_gradle_plugin_tasks.md#initializeIntelliJPlatformPlugin-coroutinesJavaAgent)


## `CustomIntelliJPlatformVersionAware`
{#CustomIntelliJPlatformVersionAware}

<tldr>

**Depends on**: [`IntelliJPlatformVersionAware`](#IntelliJPlatformVersionAware)

**Inherited by**: [`SandboxAware`](#SandboxAware), [`runIde`](tools_intellij_platform_gradle_plugin_tasks.md#runIde), [`testIde`](tools_intellij_platform_gradle_plugin_tasks.md#testIde), [`testIdePerformance`](tools_intellij_platform_gradle_plugin_tasks.md#testIdePerformance), [`testIdeUi`](tools_intellij_platform_gradle_plugin_tasks.md#testIdeUi)

**Sources**: [`CustomIntelliJPlatformVersionAware`](%gh-ijpgp%/src/main/kotlin/org/jetbrains/intellij/platform/gradle/tasks/aware/CustomIntelliJPlatformVersionAware.kt)

</tldr>

By default, the project with the IntelliJ Platform Gradle Plugin applied required the presence of the IntelliJ Platform, referred to later by various tasks, configurations, and extensions.

The custom IntelliJ Platform concept allows using another version, i.e., to run a guest IDE or tests against it.

When applying this interface to the task, custom configurations to hold new dependencies defined by [`type`](#CustomIntelliJPlatformVersionAware-type) and [`version`](#CustomIntelliJPlatformVersionAware-version) (or [`localPath`](#CustomIntelliJPlatformVersionAware-localPath), if referring to the local IntelliJ Platform instance) are created, as well as a dedicated [`prepareSandbox`](tools_intellij_platform_gradle_plugin_tasks.md#prepareSandbox) task.

Configurations, as well as the task preparing sandbox for running and testing the custom IntelliJ Platform (if required), have a random suffix applied to avoid collisions.


### `type`
{#CustomIntelliJPlatformVersionAware-type}

An input property to configure the type of the custom IntelliJ Platform.

By default, it refers to the IntelliJ Platform type used by the current project.

{style="narrow"}
Type
: [`IntelliJPlatformType`](tools_intellij_platform_gradle_plugin_types.md#IntelliJPlatformType)


### `version`
{#CustomIntelliJPlatformVersionAware-version}

An input property to configure the version of the custom IntelliJ Platform.

By default, it refers to the IntelliJ Platform version used by the current project.

{style="narrow"}
Type
: `Property<String>`


### `localPath`
{#CustomIntelliJPlatformVersionAware-localPath}

An input property to define the path to the local IntelliJ Platform instance to configure the version of the custom IntelliJ Platform.

The local path precedes the IntelliJ Platform resolution using the [`type`](#CustomIntelliJPlatformVersionAware-type) and [`version`](#CustomIntelliJPlatformVersionAware-version) properties.

{style="narrow"}
Type
: `DirectoryProperty`


## `IntelliJPlatformVersionAware`
{#IntelliJPlatformVersionAware}

<tldr>

**Inherited by**: [`CustomIntelliJPlatformVersionAware`](#CustomIntelliJPlatformVersionAware), [`RuntimeAware`](#RuntimeAware), [`SandboxAware`](#SandboxAware), [`SplitModeAware`](#SplitModeAware), [`initializeIntelliJPlatformPlugin`](tools_intellij_platform_gradle_plugin_tasks.md#initializeIntelliJPlatformPlugin), [`patchPluginXml`](tools_intellij_platform_gradle_plugin_tasks.md#patchPluginXml), [`printBundledPlugins`](tools_intellij_platform_gradle_plugin_tasks.md#printBundledPlugins), [`verifyPluginProjectConfiguration`](tools_intellij_platform_gradle_plugin_tasks.md#verifyPluginProjectConfiguration)

**Sources**: [`IntelliJPlatformVersionAware`](%gh-ijpgp%/src/main/kotlin/org/jetbrains/intellij/platform/gradle/tasks/aware/IntelliJPlatformVersionAware.kt)

</tldr>

This interface provides tasks a possibility for accessing information about the IntelliJ Platform currently used in the project.

The [`intelliJPlatformConfiguration`](#IntelliJPlatformVersionAware-intelliJPlatformConfiguration) input property receives a dependency added to the `intellijPlatform` configuration, which eventually is resolved and lets to access the IntelliJ Platform details such as [`ProductInfo`](tools_intellij_platform_gradle_plugin_types.md#ProductInfo) or the path to the IntelliJ Platform directory.

It is required to have a dependency on the IntelliJ Platform added to the project with helpers available in [](tools_intellij_platform_gradle_plugin_dependencies_extension.md).


### `intelliJPlatformConfiguration`
{#IntelliJPlatformVersionAware-intelliJPlatformConfiguration}

Holds the `intellijPlatform` configuration with the IntelliJ Platform dependency added.

It should not be directly accessed.

{style="narrow"}
Type
: `ConfigurableFileCollection`


### `platformPath`
{#IntelliJPlatformVersionAware-platformPath}

Provides a direct path to the IntelliJ Platform dependency artifact.

{style="narrow"}
Access
: Read-only

Type
: `Path`

### `productInfo`
{#IntelliJPlatformVersionAware-productInfo}

Provides information about the IntelliJ Platform product.

The information is retrieved from the <path>product-info.json</path> file in the IntelliJ Platform directory.

{style="narrow"}
Access
: Read-only

Type
: [`ProductInfo`](tools_intellij_platform_gradle_plugin_types.md#ProductInfo)


### `validateIntelliJPlatformVersion()`
{#IntelliJPlatformVersionAware-validateIntelliJPlatformVersion}

Validates that the resolved IntelliJ Platform is supported by checking against the minimal supported IntelliJ Platform version.

Invokes [`ProductInfo.validateSupportedVersion()`](tools_intellij_platform_gradle_plugin_types.md#ProductInfo-validateSupportedVersion).

{style="narrow"}
Throws
: `IllegalArgumentException`


## `JavaCompilerAware`
{#JavaCompilerAware}

<tldr>

**Inherited by**: [`instrumentCode`](tools_intellij_platform_gradle_plugin_tasks.md#instrumentCode)

**Sources**: [`JavaCompilerAware`](%gh-ijpgp%/src/main/kotlin/org/jetbrains/intellij/platform/gradle/tasks/aware/JavaCompilerAware.kt)

</tldr>

The interface provides the dependency on Java Compiler required for the [](tools_intellij_platform_gradle_plugin.md#code-instrumentation) to properly configure Ant tasks provided by the IntelliJ Platform.


### `javaCompilerConfiguration`
{#JavaCompilerAware-javaCompilerConfiguration}

Holds the `intellijPlatformJavaCompiler` configuration with the Java Compiler dependency added.

{style="narrow"}
Type
: `ConfigurableFileCollection`


## `PluginAware`
{#PluginAware}

<tldr>

**Depends on**: [`patchPluginXml`](tools_intellij_platform_gradle_plugin_tasks.md#patchPluginXml)

**Inherited by**: [`TestableAware`](#TestableAware), [`RunnableIdeAware`](#RunnableIdeAware), [`jarSearchableOptions`](tools_intellij_platform_gradle_plugin_tasks.md#jarSearchableOptions)

**Sources**: [`PluginAware`](%gh-ijpgp%/src/main/kotlin/org/jetbrains/intellij/platform/gradle/tasks/aware/PluginAware.kt)

</tldr>

This interface provides information about the currently built plugin.

It resolves and parses the final <path>plugin.xml</path> descriptor file, making its details easily accessible.


### `pluginXml`
{#PluginAware-pluginXml}

Holds the path to the patched <path>plugin.xml</path> file.

{style="narrow"}
Type
: `RegularPropertyFile`

Default value
: [`patchPluginXml.outputFile`](tools_intellij_platform_gradle_plugin_tasks.md#patchPluginXml-outputFile)


### `parse {}`
{#PluginAware-parse}

The `parse` method provides a possibility for parsing the <path>pluginXml</path> file and direct access to the [`PluginBean`](tools_intellij_platform_gradle_plugin_types.md#PluginBean) object.

Should be used along with the [`pluginXml`](#PluginAware-pluginXml) property like:

```kotlin
abstract class RetrievePluginNameTask : DefaultTask(), PluginAware

val retrievePluginName by tasks.registering(RetrievePluginNameTask::class) {
    doLast {
        val name = pluginXml.parse { name }.get()
        println("Plugin Name: $name")
    }
}
```


## `PluginVerifierAware`
{#PluginVerifierAware}

<tldr>

**Inherited by**: [`verifyPlugin`](tools_intellij_platform_gradle_plugin_tasks.md#verifyPlugin)

**Sources**: [`PluginVerifierAware`](%gh-ijpgp%/src/main/kotlin/org/jetbrains/intellij/platform/gradle/tasks/aware/PluginVerifierAware.kt)

</tldr>

The interface provides the path to the IntelliJ Plugin Verifier executable.

It is required to have a dependency on the IntelliJ Plugin Verifier added to the project with [`intellijPlatform.pluginVerifier()`](tools_intellij_platform_gradle_plugin_dependencies_extension.md) dependencies extension.


### `pluginVerifierExecutable`
{#PluginVerifierAware-pluginVerifierExecutable}

Path to the IntelliJ Plugin Verifier executable.

{style="narrow"}
Type
: `RegularFileProperty`


## `RunnableIdeAware`
{#RunnableIdeAware}

<tldr>

**Depends on**: [`AutoReloadAware`](#AutoReloadAware), [`CoroutinesJavaAgentAware`](#CoroutinesJavaAgentAware), [`PluginAware`](#PluginAware), [`RuntimeAware`](#RuntimeAware), [`SandboxAware`](#SandboxAware), [`SplitModeAware`](#SplitModeAware), [`initializeIntelliJPlatformPlugin`](tools_intellij_platform_gradle_plugin_tasks.md#initializeIntelliJPlatformPlugin)

**Inherited by**: [`buildSearchableOptions`](tools_intellij_platform_gradle_plugin_tasks.md#buildSearchableOptions), [`runIde`](tools_intellij_platform_gradle_plugin_tasks.md#runIde), [`testIde`](tools_intellij_platform_gradle_plugin_tasks.md#testIde), [`testIdePerformance`](tools_intellij_platform_gradle_plugin_tasks.md#testIdePerformance), [`testIdeUi`](tools_intellij_platform_gradle_plugin_tasks.md#testIdeUi)

**Sources**: [`RunnableIdeAware`](%gh-ijpgp%/src/main/kotlin/org/jetbrains/intellij/platform/gradle/tasks/aware/RunnableIdeAware.kt)

</tldr>

The interface which uses a set of various interfaces required for running a guest IDE.  Inherits from:
- [`CoroutinesJavaAgentAware`](#CoroutinesJavaAgentAware)
- [`PluginAware`](#PluginAware)
- [`RuntimeAware`](#RuntimeAware)
- [`SandboxAware`](#SandboxAware)
- `JavaForkOptions`


## `RuntimeAware`
{#RuntimeAware}

<tldr>

**Inherited by**: [`RunnableIdeAware`](#RunnableIdeAware), [`TestableAware`](#TestableAware), [`verifyPlugin`](tools_intellij_platform_gradle_plugin_tasks.md#verifyPlugin)

**Sources**: [`RuntimeAware`](%gh-ijpgp%/src/main/kotlin/org/jetbrains/intellij/platform/gradle/tasks/aware/RuntimeAware.kt)

</tldr>

This interface provides access to the Java Runtime (i.e., JetBrains Runtime) resolved with `RuntimeResolver`.


### `runtimeDirectory`
{#RuntimeAware-runtimeDirectory}

Java Runtime parent directory.

{style="narrow"}
Type
: `DirectoryProperty`


### `runtimeArchitecture`
{#RuntimeAware-runtimeArchitecture}

An architecture of the Java Runtime currently used for running Gradle.

{style="narrow"}
Type
: `Property<String>`


### `runtimeMetadata`
{#RuntimeAware-runtimeMetadata}

Metadata object of the Java Runtime currently used for running Gradle.

{style="narrow"}
Type
: `Property<String>`


### `runtimeLauncher`
{#RuntimeAware-runtimeLauncher}

A custom `JavaLauncher` instance configured with the resolved [`runtimeDirectory`](#RuntimeAware-runtimeDirectory).

{style="narrow"}
Type
: `Property<String>`


## `SandboxAware`
{#SandboxAware}

<tldr>

**Depends on**: [`prepareSandbox`](tools_intellij_platform_gradle_plugin_tasks.md#prepareSandbox)

**Inherited by**: [`RunnableIdeAware`](#RunnableIdeAware), [`SandboxProducerAware`](#SandboxProducerAware), [`jarSearchableOptions`](tools_intellij_platform_gradle_plugin_tasks.md#jarSearchableOptions), [`prepareSandbox`](tools_intellij_platform_gradle_plugin_tasks.md#prepareSandbox), [`verifyPluginStructure`](tools_intellij_platform_gradle_plugin_tasks.md#verifyPluginStructure)

**Sources**: [`SandboxAware`](%gh-ijpgp%/src/main/kotlin/org/jetbrains/intellij/platform/gradle/tasks/aware/SandboxAware.kt)

</tldr>

The interface provides quick access to the sandbox container and specific directories located within it.

The path to the sandbox container is obtained using the [`intellijPlatform.sandboxContainer`](tools_intellij_platform_gradle_plugin_extension.md#intellijPlatform-sandboxContainer) extension property and the type and version of the IntelliJ Platform applied to the project.

Paths respect custom IntelliJ Platform when combined with [`CustomIntelliJPlatformVersionAware`](#CustomIntelliJPlatformVersionAware).


### `sandboxSuffix`
{#SandboxAware-sandboxSuffix}

Represents the suffix used i.e., for test-related tasks.

{style="narrow"}
Type
: `Property<String>`


### `sandboxContainerDirectory`
{#SandboxAware-sandboxContainerDirectory}

The container for all sandbox-related directories.

The directory name depends on the platform type and version currently used for running a task.

{style="narrow"}
Type
: `DirectoryProperty`


### `sandboxConfigDirectory`
{#SandboxAware-sandboxConfigDirectory}

A configuration directory located within the [`sandboxContainerDirectory`](#SandboxAware-sandboxContainerDirectory).

{style="narrow"}
Type
: `DirectoryProperty`


### `sandboxPluginsDirectory`
{#SandboxAware-sandboxPluginsDirectory}

A plugins directory located within the [`sandboxContainerDirectory`](#SandboxAware-sandboxContainerDirectory).

{style="narrow"}
Type
: `DirectoryProperty`


### `sandboxSystemDirectory`
{#SandboxAware-sandboxSystemDirectory}

A system directory located within the [`sandboxContainerDirectory`](#SandboxAware-sandboxContainerDirectory).

{style="narrow"}
Type
: `DirectoryProperty`


### `sandboxLogDirectory`
{#SandboxAware-sandboxLogDirectory}

A log directory located within the [`sandboxContainerDirectory`](#SandboxAware-sandboxContainerDirectory).

{style="narrow"}
Type
: `DirectoryProperty`


## `SandboxProducerAware`
{#SandboxProducerAware}

<tldr>

**Depends on**: [`SandboxAware`](#SandboxAware)

**Inherited by**: [`prepareSandbox`](tools_intellij_platform_gradle_plugin_tasks.md#prepareSandbox)

**Sources**: [`SandboxProducerAware`](%gh-ijpgp%/src/main/kotlin/org/jetbrains/intellij/platform/gradle/tasks/aware/SandboxProducerAware.kt)

</tldr>

An interface used to distinguish between the [SandboxAware] consumers and producers.


## `SigningAware`
{#SigningAware}

<tldr>

**Inherited by**: [`signPlugin`](tools_intellij_platform_gradle_plugin_tasks.md#signPlugin), [`verifyPluginSignature`](tools_intellij_platform_gradle_plugin_tasks.md#verifyPluginSignature)

**Sources**: [`SigningAware`](%gh-ijpgp%/src/main/kotlin/org/jetbrains/intellij/platform/gradle/tasks/aware/SigningAware.kt)

</tldr>

The interface provides the path to the Marketplace ZIP Signer executable.

It is required to have a dependency on the Marketplace ZIP Signer added to the project with [`intellijPlatform.zipSigner()`](tools_intellij_platform_gradle_plugin_dependencies_extension.md) dependencies extension.


### `zipSignerExecutable`
{#SigningAware-zipSignerExecutable}

Path to the Marketplace ZIP Signer executable.

{style="narrow"}
Type
: `RegularFileProperty`


## `SplitModeAware`
{#SplitModeAware}

<tldr>

**Inherited by**: [`RunnableIdeAware`](#RunnableIdeAware)

**Sources**: [`SplitModeAware`](%gh-ijpgp%/src/main/kotlin/org/jetbrains/intellij/platform/gradle/tasks/aware/SplitModeAware.kt)

</tldr>

When you develop a plugin, you may want to check how it works in remote development mode, when one machine is running the backend part and another is running a frontend part (JetBrains Client) which connects to the backend.

This property allows running the IDE with backend and frontend parts running in separate processes.
The developed plugin is installed in the backend part.

Split Mode requires the IntelliJ Platform in the version `241.14473` or later.


## `TestableAware`
{#TestableAware}

<tldr>

**Inherited by**: [`prepareTest`](tools_intellij_platform_gradle_plugin_tasks.md#prepareTest), [`testIde`](tools_intellij_platform_gradle_plugin_tasks.md#testIde)

**Sources**: [`TestableAware`](%gh-ijpgp%/src/main/kotlin/org/jetbrains/intellij/platform/gradle/tasks/aware/TestableAware.kt)

</tldr>

Interface used to describe tasks used for running tests, such as a customizable [`testIde`](tools_intellij_platform_gradle_plugin_tasks.md#testIde) or [`prepareTest`](tools_intellij_platform_gradle_plugin_tasks.md#prepareTest) used for configuring `test` and keeping it immutable.


<include from="snippets.md" element-id="missingContent"/>
