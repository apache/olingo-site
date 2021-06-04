Title: OSGi Support

# OSGi Support

All Apache Olingo artifacts are OSGi enabled and build as bundles. Apache Olingo is using the [Apache Felix bundle plugin for Maven](http://felix.apache.org/site/apache-felix-maven-bundle-plugin-bnd.html) to generate the manifest.mf file for each bundle.

The bundles expose the complete api (`org.apache.olingo.odata2.api.*`). From the implementation part (`org.apache.olingo.odata2.core.*`) only a minimum set of packages is exposed.

There is a difficulty in relation to class loading because of Apache Olingo core bundle has to load a specific custom service implementation without having a direct dependency to it.

In Apache Olingo OData2 Version 1.0.0 we solve this issue as follow: 

### Servlet Filter in OSGi

It is assumed that an OData service implementation is deployed as a native web application (war) or as web application bundle (OSGi Enterprise Expert Group on the RFC66 Web Container specification). In both cases the Apache Olingo core bundle is not aware of this application and its classloader cannot load applications service factory by default. 

Adding this servlet filter to the service servlet configuration can solve this problem. The filter can bind the application classloader to the servlet request object, which is then used by the Apache Olingo core bundle to load applications factory class. 

~~~java
	public class ServiceFactoryFilter implements Filter {
	
	  @Override
	  public void doFilter(ServletRequest request, ServletResponse response, 
	     FilterChain chain) throws IOException, ServletException { 
	     
	    request.setAttribute(ODataServiceFactory.FACTORY_CLASSLOADER_LABEL, 
	       MyServiceFactory.class.getClassLoader()); 
	  
	     chain.doFilter(request, response); 
	  }
	}
~~~

### Extended OSGi Support

The filter approach works fine for servlets but the Apache Olingo library can be used also for different, non-servlet based scenarios.  

##### Implement own ODataApplication

A service has to implement an own `ODataApplication` and return the service factory class:


~~~java
	import org.apache.olingo.odata2.core.rest.app.AbstractODataApplication;
	
	public class CarODataApplication extends AbstractODataApplication {
	
	  @Override
	  public Class<? extends ODataServiceFactory> getServiceFactoryClass() { 
	     return CarODataServiceFactory.class; 
	  }
	}
~~~

##### Register Application in Servlet Context

Then register the application for any JAX-RS servlet implementation. Here it is the Apache CXF JAX-RS servlet.

~~~xml
	<servlet>
	  <servlet-name>CarServiceServlet</servlet-name>
	  <servlet-class>org.apache.cxf.jaxrs.servlet.CXFNonSpringJaxrsServlet</servlet-class>
	  <init-param>
	    <param-name>javax.ws.rs.Application</param-name>
	    <param-value>org.apache.olingo.odata2.sample.osgi.CarODataApplication</param-value>
	  </init-param>
	  <load-on-startup>1</load-on-startup>
	</servlet>
~~~
