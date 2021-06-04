Title: Run with Tomcat

### Run with Tomcat
Necessary steps to get your project run with [Tomcat](http://tomcat.apache.org/index.html) (tested with version `7.0.32`).

#### Required steps
  * Download Tomcat 7.0.x from [Tomcat Downloads](http://tomcat.apache.org/download-70.cgi)
  * Install Tomcat as described in [Tomcat Documentation](http://tomcat.apache.org/tomcat-7.0-doc/setup.html)
  * Start Tomcat e.g. via `$TOMCAT_HOME/bin/startup.sh` (Linux) or `$TOMCAT_HOME/bin/startup.bat` (Windows)
  * Build of OData Application
    * Build your `WAR` file of your own project
    * If you don't have a project the follow our sample project setup [here](/doc/odata2/sample-setup.html)
  * Deployment of the OData Application via simple copy of created `WAR` from the web project `./target` folder (for the sample it is `$ODATA_PROJECT_HOME/cars-web/target/olingo.odata2.sample.cars.web.war`) into `$TOMCAT_HOME/webapps`
  * After successful deployment for the sample project just open following link: http://localhost:8080/olingo.odata2.sample.cars.web to see the project entry site.