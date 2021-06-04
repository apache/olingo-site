Title: Run with Tomcat

### Run with Tomcat
Necessary steps to get your project run with [Tomcat](http://tomcat.apache.org/index.html) (tested with version `7.0.32`).

#### Required steps
  * Download Tomcat 7.0.x from [Tomcat Downloads](http://tomcat.apache.org/download-70.cgi)
  * Install Tomcat as described in [Tomcat Documentation](http://tomcat.apache.org/tomcat-7.0-doc/setup.html)
  * Start Tomcat e.g. via `$TOMCAT_HOME/bin/startup.sh` (Linux) or `$TOMCAT_HOME/bin/startup.bat` (Windows)
  * Build of OData Application
    * Build your `WAR` file of your own project
  * Deployment of the OData Application via simple copy of created `WAR` from the web project `./target` folder (for the technical scenario in version `4.0.0-beta-01` it is `$ODATA_PROJECT_HOME/lib/server-tec-svc/target/odata-server-tecsvc-4.0.0-beta-01.war`) into `$TOMCAT_HOME/webapps`
  * After successful deployment for the sample project just open following link: http://localhost:8080/odata-server-tecsvc-4.0.0-beta-01 to see the project entry site.
