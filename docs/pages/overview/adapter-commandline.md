# Running Interlok Using the Bundled Executable

The adapter executable adapter.command (Mac OSX based systems), adapter (Unix based systems), or adapter.exe (Windows) is controlled by a properties file which has the same name as the executable but with a `.lax` extension. The properties in the lax file control the initial environment and behaviour of the executable. You can make multiple copies of these files, you just need to make sure that there is an associated `.lax` file for each executable you create.

## LAX file properties ##

| Property name| Description|
|----|----|
|lax.class.path| This defines the classpath of the application separated by the platform specific separator. There should be no need to modify this property as the classpath is dynamically modified upon start-up.|
|lax.command.line.args| A list of arguments passed to the main method. These are specified in exactly the same way as they would be on the command line. For example, if you invoke your application currently as `java myApp arg1 arg2`, set this property to `arg1 arg2`. Be sure to place quotes around any arguments that have spaces. The default value of $CMD_LINE_ARGUMENTS$ indicates that any actual command-line arguments are passed in to the main method.
|lax.nl.java.option.java.heap.size.initial|This defines the initial heap size for the installer that will be invoked. This number is always specified in bytes, not in kilobytes or megabytes, and is the same as specifying the VM parameter `-Xms`.
|lax.nl.java.option.java.heap.size.max|This defines the maximum heap size in bytes for the installer that will be invoked. This number is always specified in bytes, not in kilobytes or megabytes, and is the same as specifying the VM parameter `-Xmx`.
|lax.nl.java.option.additional|The value of this property will be written to the command line verbatim. Java VM properties or settings not directly supported by the LAX configuration properties can be directly included as part of the command line used to invoke Java.
|lax.nl.java.option.debugging|Setting this to true will turn on debugging in the virtual machine|

<br/>

## Additional Windows Services ##

If you have requested it, at installation time, the adapter will install itself as a Windows service; however in some situations you might want to have additional adapters running as services on the same machine. To do this use the built-in `SC` command to add the adapter as a service under a different name.

First of all make a copy of the .exe and .lax file and give it a new name (`myAdapter.exe` and `myAdapter.lax` for example). Then we can use the _sc_ command to add to our list of services. The `-zglaxservice` parameter is important as this allows the executable to be controlled via the standard Windows service instructions.

```
sc create "myServiceName" binPath="\path\to\myAdapter.exe -zglaxservice myServiceName" DisplayName= "My Friendly Adapter Service Name"
```

It's simpler to make sure that `myServiceName` doesn't have any spaces (you can have spaces, but it is a pain to type on the commandline); other options can be used to make the adapter dependent on other services (e.g. SonicMQ); that is beyond the scope of this document.

## Running directly from the commandline ##

When wrapping the Interlok runtime as part of another script, it is sometimes preferable to directly start the Java virtual machine directly rather than using the standard executable. Provided the current working directory is the base directory of the installation, the adapter can simply be started using the standard `-jar` directive (since _3.6.5_ use `java -jar lib/interlok-boot.jar`, for previous versions using `java -jar lib/adp-core.jar`), passing in any JVM properties and command-line arguments as appropriate. If the JVM classpath has been set manually prior to starting the adapter, then note that the `-jar` directive forces the JVM to ignore any classpath that might have been specified. If you???re happy with the manual classpath then use `com.adaptris.core.management.SimpleBootstrap` as the main class. This class will not cause any additional classpath initialisation to happen (i.e. `./config`, `./lib/*.jar` and `lib/*.zip` are not added to the classpath).

?> **NOTE** since 3.6.5; in readiness for java 9; there is a new interlok-boot.jar which should be used in preference to adp-core.jar when running directly from the commandline.

!> **IMPORTANT** since 3.8.0; adp-core has been renamed to interlok-core


## Commandline Switches ##

It is possible to invoke the adapter with a number of different switches. These switches will cause the adapter to output some debugging information and then terminate.

|Switch | Description |
|----|----|
|-version| Prints out the version number of Interlok along with any optional sub-components|
|-configtest| Parses the property file, and the associated configuration to be unmarshalled. It also checks that the license is valid for the configuration. It does not check that the configuration is valid (e.g. Broker passwords) for what you're trying to do, it just asserts that the configuration is syntactically correct|

<br/>

```
$ java -jar lib/interlok-boot.jar -version
Bootstrap of Interlok 3.6-SNAPSHOT(2017-02-22) complete
Version Information
  Base Interlok: 3.6-SNAPSHOT(2017-02-22)
  Interlok Annotation Support: 3.6-SNAPSHOT(2017-02-22)
  Interlok Common Classes: 3.6-SNAPSHOT(2017-02-22)
  Interlok Config Pre-Processor: variable substitution support: 3.6-SNAPSHOT(2017-02-22)
  Interlok Config Pre-Processor: xinclude support: 3.6-SNAPSHOT(2017-02-22)
  Interlok Logging components: 3.6-SNAPSHOT(2017-02-22)
```

```
$ java -jar lib/interlok-boot.jar -configcheck bootstrap.properties
Bootstrap of Interlok 3.6-SNAPSHOT(2017-02-22) complete
TRACE [main] [BootstrapProperties] Properties resource is [bootstrap.properties]
TRACE [main] [PropertyResolver] Parsing PropertyResolver URL [jar:file:/C:/Users/lchan/work/runtime/v3-nightly/lib/adp-core.jar!/META-INF/com/adaptris/core/management/properties/resolver]
TRACE [main] [PropertyResolver] Registered Decoders : {password=com.adaptris.core.management.properties.PasswordDecoder}
TRACE [main] [BootstrapProperties] Adding org.jruby.embed.localcontext.scope=threadsafe to system properties
TRACE [main] [BootstrapProperties] Adding org.jboss.logging.provider=slf4j to system properties
INFO  [main] [RuntimeVersionControlLoader] No version control systems found.
INFO  [main] [RuntimeVersionControlLoader] No version control systems found.
TRACE [main] [RuntimeInfoComponentFactory] No RuntimeInfoComponent for class com.adaptris.core.NoRetries
TRACE [main] [RuntimeInfoComponentFactory] No RuntimeInfoComponent for class com.adaptris.core.NullProcessingExceptionHandler
TRACE [main] [RuntimeInfoComponentFactory] No RuntimeInfoComponent for class com.adaptris.core.NullLogHandler
INFO  [main] [UnifiedBootstrap] Adapter created
Config check only; terminating
```
