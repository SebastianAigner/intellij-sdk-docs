<!-- Copyright 2000-2024 JetBrains s.r.o. and contributors. Use of this source code is governed by the Apache 2.0 license. -->

# Dynamic Plugins

<link-summary>Making a plugin dynamic allows installing, updating, and uninstalling it without IDE restart, as well as hot reloading plugin changes during the development.</link-summary>

Starting with the **2020.1** release, installing, updating, and uninstalling plugins without restarting the IDE is available in the IntelliJ Platform.

During plugin development, [Auto-Reload](ide_development_instance.md#enabling-auto-reload) also allows code changes to take effect immediately in the sandbox IDE instance.
To test whether dynamic installation works correctly, verify installing [local build distribution](publishing_plugin.md#building-distribution) succeeds (see [Troubleshooting](#troubleshooting)).

Please note that any unloading problems in a production environment will simply ask the user to restart the IDE.

> If a plugin _requires_ restart (e.g., due to using native libraries) specify `require-restart="true"` for [`<idea-plugin>`](plugin_configuration_file.md#idea-plugin) root tag in <path>[plugin.xml](plugin_configuration_file.md)</path>.
>
{style="note"}

> 3rd-party **paid** plugins cannot be installed, updated or uninstalled without restarting the IDE.
>
{style="warning"}

## Restrictions

For a plugin to support this, all restrictions listed below must be met.
To verify a plugin locally, invoke <ui-path>Code | Analyze Code | Run Inspection by Name...</ui-path> and run <control>Plugin DevKit | Plugin descriptor | Plugin.xml dynamic plugin verification inspection</control> inspection on all plugin descriptor files.

For plugins hosted on the [JetBrains Marketplace](https://plugins.jetbrains.com) the built-in [Plugin Verifier](https://blog.jetbrains.com/platform/2018/07/plugins-repository-now-integrates-with-the-plugin-verification-tool/) will run these checks automatically.
See [](verifying_plugin_compatibility.md#plugin-verifier) for more information on how to run it locally or on CI.

### No Use of Components

No Components must be used; existing ones [must be migrated](plugin_components.md) to services, extensions, or listeners.

### Action Group Requires ID

All [`<group>`](plugin_configuration_file.md#idea-plugin__actions__group) elements must declare a unique `id`.

### Use Only Dynamic Extensions

Whether defined in the platform itself ([](extension_point_list.md)) or coming from other plugins, all used extension points must be marked explicitly as dynamic (see next paragraph).

Some deprecated extension points (e.g., `com.intellij.configurationProducer`) are intentionally non-dynamic, and their usage should be migrated to the corresponding replacement.

### Mark Extension Points as Dynamic

If a plugin defines its own custom extension points, they must adhere to specific usage rules and then [be declared](plugin_extension_points.md#dynamic-extension-points) ready for dynamic use explicitly.

### Configurables Depending on Extension Points

Any [`Configurable`](%gh-ic%/platform/ide-core/src/com/intellij/openapi/options/Configurable.java) which depends on dynamic extension points must implement `Configurable.WithEpDependencies`.

### No Use of Service Overrides

Application, project, and module [services](plugin_services.md) declared with `overrides="true"` are not allowed.

## Code

> Loading and unloading plugins happens in EDT and under write action.
>
{style="note"}

### `CachedValue`

Loading/Unloading a plugin clears all cached values created using [`CachedValuesManager`](%gh-ic%/platform/core-api/src/com/intellij/psi/util/CachedValuesManager.java).

### Do not Store PSI

Do not store references to PSI elements in objects which can survive plugin loading or unloading; use [`SmartPsiElementPointer`](%gh-ic%/platform/core-api/src/com/intellij/psi/SmartPsiElementPointer.java) instead.

### Do not Use `FileType`/`Language` as Map Key

Replace with `String` from `Language.getID()`/`FileType.getName()` (use inspection <control>Plugin DevKit | Code | Map key may leak</control>).

### Plugin Load/Unload Events

Register [`DynamicPluginListener`](%gh-ic%/platform/core-api/src/com/intellij/ide/plugins/DynamicPluginListener.kt) [application listener](plugin_listeners.md) to receive updates on plugin load/unload events.

This can be used to e.g., cancel long-running activities or disallow unload due to ongoing processes.

### Resource Cleanup

Use [](plugin_services.md) implementing [`Disposable`](%gh-ic%/platform/util/src/com/intellij/openapi/Disposable.java) and perform cleanup in `Disposable.dispose()`.

## Troubleshooting

When a plugin is being uninstalled or updated, the IDE waits synchronously for plugin unload and asks for restart if the unloading failed.

If it fails:

1. Try a newer version of the IDE (eventually latest available from [Early Access Program](https://eap.jetbrains.com)), in some cases platform bugs might be an issue.
   See this [list of known platform issues](https://youtrack.jetbrains.com/issues/IDEA?q=%23dynamic-plugins%20) related to handling dynamic plugins.

2. Try in a fresh and new configuration (e.g., clean the [sandbox](ide_development_instance.md#the-development-instance-sandbox-directory) or use a different configuration directory).

### Logging

All events are tracked under `com.intellij.ide.plugins.DynamicPlugins` category in IDE log file.
If a plugin fails to reload, the log will contain a cause as to why.

### Diagnosing Leaks

<procedure title="Finding leaks preventing unload">

1. Verify that the IDE is running with the VM parameter `-XX:+UnlockDiagnosticVMOptions`. When using [Gradle](configuring_plugin_project.md), specify `runIde.jvmArgs += "-XX:+UnlockDiagnosticVMOptions"` otherwise [Configure JVM Options](https://www.jetbrains.com/help/idea/tuning-the-ide.html#procedure-jvm-options).
2. Set Registry key `ide.plugins.snapshot.on.unload.fail` to `true` (Go to <ui-path>Navigate | Search Everywhere</ui-path> and type `Registry`).
3. Trigger the plugin reload.
4. Open the <path>.hprof</path> memory snapshot generated in user home directory on plugin unload, look for the plugin ID string. [IntelliJ Ultimate](https://www.jetbrains.com/help/idea/analyze-hprof-memory-snapshots.html) can open memory snapshots directly.
5. Find the `PluginClassLoader` referencing the plugin ID string
6. Look at references to the `PluginClassLoader` instance.
7. Every one of them is a memory leak (or part of a memory leak) that needs to be resolved.

</procedure>

When you've completed step 1 and 2, the log will contain more information about the memory leak, for instance the following shows a chain of field references that
is keeping the class loader in memory.

```text
2020-12-26 14:43:24,563 [ 251086]   INFO - lij.ide.plugins.DynamicPlugins - Snapshot analysis result: Root 1:
  ROOT: Global JNI
  sun.awt.X11.XInputMethod.clientComponentWindow
  com.intellij.openapi.wm.impl.IdeFrameImpl.rootPane
  com.intellij.openapi.wm.impl.IdeRootPane.myToolbar
  com.intellij.openapi.actionSystem.impl.ActionToolbarImpl.myVisibleActions
  java.util.ArrayList.elementData
  java.lang.Object[]
  com.example.ActionExample.<class>
  com.example.ActionExample.<loader>
* com.intellij.ide.plugins.cl.PluginClassLoader
```
