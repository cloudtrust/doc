# Tracking 

[Sentry](https://sentry.io/welcome/) is an open source error tracking system. It helps monitoring crashes and errors in real time.

## Sentry
By default, the error tracking is deactivated. To set it up, add the sentry-dsn to the configuration file.

Note: To obtain the sentry Data Source Name (DSN) you need to set up a project in Sentry (see [documentation](https://docs.sentry.io/quickstart/#configure-the-dsn)). 

Key | Description | Default value 
--- | ----------- | ------------- 
sentry-dsn | Sentry Data Source Name | ""