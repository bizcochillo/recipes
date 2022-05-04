 To install Jenkins and put to run in a port other than 8080 (httpPort) and put the root in other folder (webroot)

```console
java -jar jenkins.war --httpPort=PORT_NUMBER --webroot=WEBROOT_PATH
```

For instance (with username `admin/admin`), pick the admin password, install the recommended plugins
```console
java -jar jenkins.war --httpPort=18080
```
- Install git
- Install maven (unzip in a folder and set the extracted path to two environment variables `M2_HOME` and `MAVEN_HOME` and add a path in `PATH` variable pointing to `%M2_HOME\bin`
