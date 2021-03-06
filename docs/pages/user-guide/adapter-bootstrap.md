When Interlok is started without any commandline parameters, it expects to find a file `bootstrap.properties` on the classpath (generally in the _config_ directory). If a file is specified on the commandline then this is expected to be a file present on the classpath or a fully qualified filename. This properties file contains the initial behaviour of Interlok and has a number of settings that can be modified to suit your environment. The keys themselves tend to be treated in a case insenstive fashion (not always guaranteed) so you should try and be consistent in your naming.

## Standard properties

The list of keys and a brief description of each key is described below :

| Name/prefix | Description |
|----|----|
| adapterConfigUrl | This is an primary means of locating adapter runtime configuration; use any supported URL scheme, specify a `file:///` scheme for a local file|
| ~~beautifyXStreamOutput~~ | __Removed in 3.8.2__: ~~Defaults to false, and if true, means that an attempt is made to remove all `class=` attributes, and to only use the raw alias as the element name. this means that where the same interface is used (e.g. for `produce-connection` + `consume-connection`, order becomes important).~~ |
| configManager | Defaults to [XStreamConfigManager][]; you should never have to change this|
| enableLocalJndiServer| defaults to false, and should be set to true, if you intend on having [shared components](/pages/user-guide/adapter-jndi-guide) accessible via the `adapter:` scheme|
| httpEnableProxyAuth | Defaults to true, which means that a custom [Authenticator][] is inserted to authenticate against requests made that are of the type _RequestorType.PROXY_. The custom authenticator takes the username from one of the system properties `http.proxyUser`,`https.proxyUser`,`proxyUser` and the password from `http.proxyPassword`, `https.proxyPassword`, `proxyPass`. This can override certain behaviours available by default in your JDK (for instance, if you have a separate username/password combination for HTTP / HTTPS proxies)|
| jetty. | Properties prefixed by this key will be converted passed into to the embedded jetty instance (since 3.6.0) |
| ~~log4j12Url~~ | __Removed in 3.9.0__ : use loggingUrl instead |
| loggingConfigUrl | since 3.1.0 - If specified then then an attempt is made to configure the logging (log4j1.2 or log4j2) subsystem with the referenced URL; if this is not configured then logging initialisation uses the standard defaults for log4j. |
| managementComponents | a list of `:` separated management components that will be started.
| marshallerOutputType | The default output type for the marshaller; this defaults to XML and you should never have to modify this|
| preProcessors | A `:` separated list of [pre-processors](/pages/advanced/advanced-configuration-pre-processors) that need to be applied before configuration is unmarshalled |
| sysprop. | Properties prefixed by this key will be converted into system properties at startup (minus the `sysprop.`) |
| startAdapterQuietly | Defaults to true, and if false, then if an adapter fails to start, then the entire JVM will be terminated. |
| useJavaLangManagementFactory | Defaults to true, and you should never have to change this |
| ~~validateConfig~~ | __Removed in 3.11.0__ : Use `-configcheck` instead ~~Defaults to false, if true, then `javax.validation` style annotations will additionally be used to validate the configuration.~~ |
| operationTimeout | How long to wait for the adapter to start (in milliseconds) before giving up and bailing; defaults to 2 minutes. |

### Overriding keys using system properties

You can override a select number of properties using standard system properties

| System Property | Bootstrap key | Notes |
|----|----|----|
| interlok.config.url | adapterConfigUrl | _since 3.6.2_ `-Dinterlok.config.url=/path/to/my/adapter.xml` will cause _/path/to/may/adapter.xml_ to be the config url overriding the existing key |
| interlok.logging.url | loggingConfigUrl | _since 3.6.2_ `-Dinterlok.logging.url=/path/to/my/log4j2.xml` to specify the logging configuration |
| interlok.jmxserviceurl | jmxserviceurl | _since 3.6.2_ `-Dinterlok.jmxserviceurl=service:jmx:jmxmp://localhost:5555` will override the existing `jmxserviceurl` if the jmx management component is enabled|
| interlok.mgmt.components | managementComponents| _since 3.6.2_ `-Dinterlok.mgmt.components=jmx:jetty` will override the existing `managementComponents` setting |
| adp.license.location | | `-Dadp.license.location=/path/to/my/license.properties` overrides the license location (normally `license.properties` on the classpath) if you are using an optional component that requires it |
| interlok.license.key | | _since 3.6.2_ `-Dinterlok.license.key=ABCDEFG` provides the actual license key if you are using an optional component that requires it |

## Keys in detail.

Some of the keys deserve a bit more explanation.

### Config Synchronisation ###

The `adapterConfigUrl` key is also used a prefix for any number of keys which will enable two additional features.

- Multiple sources for the adapter runtime config.
- Synchronisation of the secondary sources with the primary source.

If many keys are prefixed by `adapterConfigUrl`, then the following rules will apply:

- The keys will be sorted using their natural order (alphabetically) prior to use.
- The first entry in the sorted list is considered the primary for synchronisation purposes; the others are considered secondary URLs.
- Only local file system URLs (file:///) will be considered candidates for secondary synchronisation.

Which means that you can configure something like :

```properties
adapterConfigUrl=http://localhost/adapter.xml
adapterConfigUrl.1=http://localhost/adapter2.xml
adapterConfigUrl.2=file:///./config/adapter3.xml
```

`http://localhost/adapter.xml` is considered the primary source for the adapter runtime object. If that is not available then `http://localhost/adapter2.xml` is used. If the primary source is available, then it will be synchronised to `file:///./config/adapter3.xml` only. Synchronisation occurs after the Adapter is un-marshalled; but before any initialisation of components occurs. The entire adapter is marshalled again to the secondary source(s) which means that it is always written out as a monolithic XML document.

### Management Components ###

Management components are Interlok components that exist outside of the normal adapter lifecycle. Typical examples of this are the JMX component, ActiveMQ components and the embedded Jetty component which hosts the UI. The `:` separated list here can be either a short name (e.g. `jetty`) or a fully qualified classname of something that implements [ManagementComponent][]

#### JMX Component ####

!> **WARNING** By default the JMXMP connector is run without authentication enabled.  For production environments it is highly recommended you enable JMXMP authentication as documented [here](/pages/advanced/advanced-jmx).  Further information on the vulnerabilities of un-protected JMX can be found in [this blog post](https://www.acunetix.com/blog/web-security-zone/old-java-libraries/) by Aleksei Tiurin

If the JMX management component is specified via `managementComponents=jmx` then additional keys in bootstrap.properties determine the behaviour of the component.

| Name/prefix | Description |
|----|----|
|jmxserviceurl|The URL that will be passed into `JMXConnectorServerFactory#newJMXConnectorServer()`|
|jmxserviceurl.objectname| The ObjectName to be associated with the `JMXConnectorServer` when registering it as an MBean, defaults to `Adaptris:type=JmxConnectorServer`|
|jmxserviceurl.env.| Each property that matches this prefix is passed through to the JMXConnectorServer as part of its environment (minus the prefix); if the JMXConnectorServer required specific configuration, this is where you would do it. e.g. `jmxserviceurl.env.myEnvironment=ABCDE` would cause an environment containing `myEnvironment=ABCDE` to be passed into `JMXConnectorServerFactory#newJMXConnectorServer()`. The environment will be the equivalent of `Map<String, String>()`.

So, if we wanted to enable JMX over JMS using SonicMQ then we could have :

```properties
managementComponents=jmx
jmxserviceurl=service:jmx:sonicmq:///tcp://localhost:2506
jmxserviceurl.env.jmx.brokerUser=Administrator
jmxserviceurl.env.jmx.brokerPassword=Administrator
jmxserviceurl.env.jmx.type=Topic
jmxserviceurl.env.jmx.destination=SampleQ4
```

#### ActiveMQ Component ####

Since 3.6.0

If the ActiveMQ management component is specified via `managementComponents=activemq` and if you have downloaded the [interlok-activemq][] component, including the required dependencies into your Interlok lib directory, then upon Interlok start-up an ActiveMQ broker will also be started.

You can supply your own ActiveMQ configuration by setting the following property `activemq.config.filename` in your bootstrap.properties file. Simply set the value of this property to be the exact name of your ActiveMQ configuration file (which of course will need to be on the Interlok classpath, typically in your Interlok/config directory). Should you choose not to supply your own configuration file a default minimal configuration will be applied (found packaged in the interlok-activemq.jar);

```xml
<beans
  xmlns="http://www.springframework.org/schema/beans"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
  http://activemq.apache.org/schema/core http://activemq.apache.org/schema/core/activemq-core.xsd">
    <broker xmlns="http://activemq.apache.org/schema/core" brokerName="defaultMgmtComponentBroker" dataDirectory="activemq-data/defaultMgmtComponentBroker">
        <destinationPolicy>
            <policyMap>
              <policyEntries>
                <policyEntry topic=">" >
                  <pendingMessageLimitStrategy>
                    <constantPendingMessageLimitStrategy limit="1000"/>
                  </pendingMessageLimitStrategy>
                </policyEntry>
              </policyEntries>
            </policyMap>
        </destinationPolicy>
        <managementContext>
            <managementContext createConnector="false"/>
        </managementContext>
        <persistenceAdapter>
            <kahaDB directory="activemq-data/defaultMgmtComponentBroker-kahadb"/>
        </persistenceAdapter>
        <systemUsage>
          <systemUsage>
             <memoryUsage>
               <memoryUsage percentOfJvmHeap="70" />
             </memoryUsage>
             <storeUsage>
               <storeUsage limit="10 gb"/>
             </storeUsage>
             <tempUsage>
               <tempUsage limit="5 gb"/>
             </tempUsage>
          </systemUsage>
        </systemUsage>
        <transportConnectors>
            <transportConnector name="openwire" uri="tcp://0.0.0.0:61616?maximumConnections=1000&amp;wireFormat.maxFrameSize=104857600"/>
            <transportConnector name="amqp" uri="amqp://0.0.0.0:5672?maximumConnections=1000&amp;wireFormat.maxFrameSize=104857600"/>
            <transportConnector name="stomp" uri="stomp://0.0.0.0:61613?maximumConnections=1000&amp;wireFormat.maxFrameSize=104857600"/>
            <transportConnector name="mqtt" uri="mqtt://0.0.0.0:1883?maximumConnections=1000&amp;wireFormat.maxFrameSize=104857600"/>
        </transportConnectors>
        <shutdownHooks>
            <bean xmlns="http://www.springframework.org/schema/beans" class="org.apache.activemq.hooks.SpringContextHook" />
        </shutdownHooks>
    </broker>
</beans>
```

JMS connections within your Interlok workflows will be able to access the broker with the URL connection strings of either;

- vm://defaultMgmtComponentBroker?create=false
- tcp://localhost:61616

__Note:__ Sometimes the ActiveMQ broker can take a few seconds to fully start-up and initialize, therefore connection errors may be logged by Interlok during startup if you are using ActiveMQ as part of a channel/workflow.  Simply wait for the connections to be re-established.

#### Apache Artemis ####

Since 3.9.0

Much the same as the ActiveMQ component above, you can instead launch a JMS 2.0 broker if the Artemis management component is specified via `managementComponents=artemis` and if you have downloaded the [interlok-mgmt-artemis][] component, including the required dependencies into your Interlok lib directory, then upon Interlok start-up an Apache Artemis broker will also be started.

You can supply your own Artemis configuration by setting the following property `activemq.config.filename` in your bootstrap.properties file. Simply set the value of this property to be the exact name of your Artemis broker configuration file (which of course will need to be on the Interlok classpath, typically in your Interlok/config directory). Should you choose not to supply your own configuration file a default minimal configuration will be applied which will have all security disabled.

Connections to the locally running broker use JNDI;

```xml
<vendor-implementation class="standard-jndi-implementation">
  <jndi-params>
	<key-value-pair>
	  <key>java.naming.factory.initial</key>
	  <value>org.apache.activemq.artemis.jndi.ActiveMQInitialContextFactory</value>
	</key-value-pair>
	<key-value-pair>
	  <key>java.naming.provider.url</key>
	  <value>tcp://localhost:61616?type=CF</value>
	</key-value-pair>
  </jndi-params>
  <jndi-name>ConnectionFactory</jndi-name>
  <extra-factory-configuration class="no-op-jndi-factory-configuration"/>
</vendor-implementation>
```

#### Jetty Component ####

In some systems the temp directory is periodically purged, this can stop the GUI from serving it's pages. There are two ways in which to handle this;

- The simple solution is to stop and restart your adapter's instance this will recreate the necessary WAR directories for the GUI to run.

- The better and more stable solution is changing the location in which the WAR directories are generated. The following will explain in the simlplest way how to achieve this.

You will need to begin by creating the directory that you would your WAR to be generated in from now on.
In the adapter's config directory locate and open the `jetty.xml` file and locate the `jetty.home` property within the `org.eclipse.jetty.deploy.providers.WebAppProvider` class tag below that property insert the following:

`<Set name="tempDir"><Property name="jetty.home" default="." />`__${/desired/temp/path}__`</Set>`

This will set your new path to '/desired/temp/path' for the GUI's WAR directories finally before starting the adapter create the directories for the WARs to be generated in.

---


If jetty is enabled via `managementComponents=jetty` then an additional key is required : `webServerConfigUrl`. This should contain the fully qualified filename for a jetty configuration file. As the UI requires the jetty component and communicates with Adapters using JMX, then if you intend on using the UI you should always have `managementComponents=jmx:jetty`.

```properties
managementComponents=jetty:jmx
webServerConfigUrl=./config/jetty.xml
```

?> **TIP** since 3.6.0 then system properties prefixed with `jetty.` will added as config properties to the embedded jetty. Properties defined in _bootstrap.properties_ will always take precedence.

#### SSH Tunnel ####

Since 3.7.1

We don't envisage you using the sshtunnel management component in production (if you need to, then we'd suggest you probably need to talk to your network team); however it can be useful to temporarily run an adapter locally that uses a tunnel to connect to a remote services. More documentation is available on the [interlok-sshtunnel][] github project page

#### Exec Component ####

Since 3.8.2

[interlok-exec][] allows you to startup up arbitrary programs as part of the bootstrap of Interlok. Bear in mind that Interlok isn't really a process manager, and you should always consider using the appropriate tool for the job (e.g. systemd to startup your external program as a service). However, if that isn't possible then [interlok-exec][] can start processes which are subsequently managed as part of the Interlok instance lifecycle. When enabled executables are grouped using an identifier in the form `exec.IDENTIFIER.cmd`; this is easiest to illustrate with an example:

```properties
managementComponents=jmx:exec:jetty

exec.activemq.working.dir=/home/vagrant/activemq
exec.activemq.start.command=./bin/activemq.sh start
exec.activemq.stop.command=./bin/activemq.sh stop
exec.activemq.process.monitor.ms=10000
exec.activemq.process.debug=true

exec.tomcat.working.dir=/home/vagrant/tomcat
exec.tomcat.start.command=./bin/catalina.sh start
exec.tomcat.process.monitor.ms=10000
exec.tomcat.process.debug=true

```

In this instance there are two executable groups configured so

- Upon start we execute `catalina.sh start` and `activemq.sh start` respectively. The working directories for those processes are `/home/vagrant/tomcat` and `/home/vagrant/activemq` respectively.
  - Any output to standard error/output will be redirected at TRACE level to the standard interlok logfile
- Every 10 seconds, we check the process to see if they are alive
  - Because process.debug is true, then you will get logging in any configured log file at trace level for the process monitoring
  - If the process is dead, than we attempt to restart the executable.
- Upon interlok shutdown, the script `activemq.sh stop` will be executed for the _activemq_ exec group only.

#### Exposing workflows via a REST interface ####

Since 3.8.3 : see [Exposing Workflows as a RESTful service](/pages/user-guide/adapter-hosting-rest)  for more details.

### System Properties ###

Properties prefixed by `sysprop.` (note the `.`) will be converted into system properties at boot time (minus the prefix). For instance specifying `sysprop.myEnvironment=ABCDE` will be equivalent to using `-DmyEnvironment=ABCDE` on the command-line. These will overwrite any system properties that you may have already specified on the command-line. Use one method or the other, don???t mix the two.

Sensitive system property values may be stored encoded in the file; they will be decoded at boot time and the decoded value used for System.setProperty(). Of course, this means these values still plain text within the JVM, but are encoded for the purposes of storage on the file system in configuration/startup scripts. The syntax for an encoded property is to use {password} at the start of the property value; for instance: `sysprop.myEncodedString={password}PW:AA...N` (skipped some actual characters) is functionally equivalent to specifying ???DmyEncodedString=admin on the command-line. These sensitive values will have been encrypted using the [password handling mechanism][advanced-password-handling].

If you were using JRuby, and you wanted to ensure that variable scope was threadsafe; and you needed to specify a javax.net.ssl keystore and password; the keystore password is of course sensitive so you have encrypted it.

```properties
sysprop.org.jruby.embed.localcontext.scope=threadsafe
# Not all the text shown here for brevity.
sysprop.javax.net.ssl.keyStorePassword={password}PW:AA...N
sysprop.javax.net.ssl.keyStore=/path/to/my/keystore
```

!> **IMPORTANT** Although system properties specified `bootstrap.properties` are set before any other initialisation occurs; it may be too late for some system properties. The behaviour is not guaranteed.

### Pre Processors ###

Pre-Processors are components that allow you to inject some additional processing of Interlok configuration files before attempting to unmarshal them. It is discussed in more detail in the [pre-processors](/pages/advanced/advanced-configuration-pre-processors) document.

[XStreamConfigManager]: https://nexus.adaptris.net/nexus/content/sites/javadocs/com/adaptris/interlok-core/3.11-SNAPSHOT/com/adaptris/core/management/config/XStreamConfigManager.html
[ManagementComponent]: https://nexus.adaptris.net/nexus/content/sites/javadocs/com/adaptris/interlok-core/3.11-SNAPSHOT/com/adaptris/core/management/ManagementComponent.html
[Authenticator]: http://docs.oracle.com/javase/7/docs/api/java/net/Authenticator.html
[interlok-mgmt-artemis]: https://nexus.adaptris.net/nexus/content/groups/public/com/adaptris/interlok-artemis/
[interlok-sshtunnel]: https://github.com/adaptris/interlok-sshtunnel
[interlok-exec]: https://github.com/adaptris/interlok-exec
