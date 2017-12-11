# Keycloak Sentry integration

Sentry is an open source error traking software that publishes errors and logs to a network server. It's [website](https://sentry.io/welcome/) states that one should "Stop hoping that users will report errors". 

Sentry allows for an integration with a large number of platforms, including Java, the language used in Keycloak, Their [usage documentation](https://docs.sentry.io/clients/java/usage/) states that while it is possible to use Sentry manually, it is highly recommended to use one of the provided integrations. These integrations are basically common frameworks like Android of Spring, or popular logging engines such as log4j, Logback or JUL.

## Implementation in Sentry
The installation of Sentry is beyond the scope of this document. But once it is installaled and correctly set up, simply:

1. Create a new project
2. Select java from choice of framework, and select a project name (for example keycloak)
3. Go to your new project and open the project settings
4. Go to the **Client Keys (DSN)** section.
5. Note the DSN, This value will later have to be entered in keycloak's configuration.

## Implementation in Keycloak

Of the loggers supported by Sentry, Keycloak supports, via jboss, log4j, log4j2 and JUL, as seen in `org.jboss.logging`, but by defaut log4j seems to be used. The [Sentry documentation for log4j](https://docs.sentry.io/clients/java/modules/log4j/) states that a specific appender, `io.sentry.log4j.SentryAppender` must be used to send the information to Sentry.

In Keycloak this basically means:

* Adding the sentry and sentry-log4j libraries to Keycloak modules. Any loaded module can be used, including one specifically for Sentry (in which case do not forget to add the sentry module to the `layers.conf`).
* Adding the appender to the standalone.xml (or configuration xml used).
* Adding the Sentry DSN (basically the address of the Sentry server) to the same configuration file as a Java system property. There are other methods to do this (such as using a system environment, see [relevent documentation](https://docs.sentry.io/clients/java/config/)), but the system property is the best for Keycloak,

The directory structure is the following, where the base directory is the base directory of the module (in our case `<keycloak root>/modules/system/layers/sentry`):

```
.
`-- io
    `-- sentry
        |-- log4j
        |   `-- main
        |       |-- module.xml
        |       `-- sentry-log4j-1.6.3.jar
        `-- main
            |-- module.xml
            `-- sentry-1.6.3.jar
```

The module.xml for the base sentry jar is as follows:

```xml
<?xml version="1.0" ?>
<module xmlns="urn:jboss:module:1.1" name="io.sentry">

    <resources>
        <resource-root path="sentry-1.6.3.jar"/>
    </resources>

    <dependencies>
        <module name="com.fasterxml.jackson.core.jackson-core"/>
		<module name="org.slf4j"/>
		<module name="javax.api"/>
    </dependencies>
</module>
```

And for the sentry-log4j;

```xml
<?xml version="1.0" ?>
<module xmlns="urn:jboss:module:1.1" name="io.sentry.log4j">

    <resources>
        <resource-root path="sentry-log4j-1.6.3.jar"/>
    </resources>

    <dependencies>
        <module name="io.sentry"/>
		<module name="org.apache.log4j"/>
    </dependencies>
</module>
```

The rest of the configuration can be done by running the following three lines in a connected jboss-cli, all three of which simply modify the configuration xml (standalone.xml in our case).

To set the defintion for the SentryAppender:
```
/subsystem=logging/custom-handler=sentry:add(name=sentry,class=io.sentry.log4j.SentryAppender,module=io.sentry.log4j,enabled=true,formatter=PATTERN,level=WARN)
```
This adds the following lines to the configuration

```xml
...
    <profile>
        <subsystem xmlns="urn:jboss:domain:logging:3.0">
        ...
        <custom-handler name="sentry" class="io.sentry.log4j.SentryAppender" module="io.sentry.log4j">
                <level name="WARN"/>
                <formatter>
                    <pattern-formatter pattern="PATTERN"/>
                </formatter>
            </custom-handler>
            ...
```

To add the SentryAppender to the root logger:
```
/subsystem=logging/root-logger=ROOT:add-handler(name=sentry)
```
This adds the following lines to the configuration

```xml
...
            <root-logger>
                <level name="INFO"/>
                <handlers>
                    ...
                    <handler name="sentry"/>
                </handlers>
            </root-logger>
            ...
```

To add the DSN to as a system property:
```
/system-property=sentry.dsn:add(<DSN value>)
```
This adds the following lines to the configuration

```xml
<server xmlns="urn:jboss:domain:5.0">
...
    <system-properties>
        <property name="sentry.dsn" value="replace with DSN Value"/>
    </system-properties>
    ...
```
