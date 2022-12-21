Title:     Project setup for the OData library for JavaScript

# Project setup          

### Software requirements

* Git client 
* NodeJS installation (>= 0.8.0)
* Java VM ( only for running the Relase audit checks)

Please install this software and check the correct installation with help of the following commands which 
should produce a similar output on your system:

    $ git --version
    git version 1.8.3.msysgit.0
    
    $ node --version
    v0.11.13

    $ npm --version
    1.4.6
    
    $ java -version
    java version "1.7.0_65"
    Java(TM) SE Runtime Environment (build 1.7.0_65-b20)
    Java HotSpot(TM) Client VM (build 24.65-b04, mixed mode, sharing)


### Install global node.js packages

* grunt client
  The grunt client searches the current directory for a grunt configuration file and starts the
  Grunt installation next the this configuration file. Installation via  
    ``npm install -g grunt-cli``  
  
    Test via

        $ grunt -version
        grunt-cli v0.1.13
   
### Download source

Open your (git-)bash and use the following commonand:

``git clone https://gitbox.apache.org/repos/asf/olingo-odata4-js``

### Install the project dependencies

##### JavaScript #####

After downloading navigate into the project folder via

``cd olingo-odata4-js/``

and install the dependencies via npm (node package manager) with

``npm install``

npm will check all dependencies listed in the package.json file and install them recursively.

If you are working from behind a proxy please set the proxy configaration for npm accordingly

  ``
  npm config set proxy http://proxy.company.com:8080
  ``
  
  ``
  npm config set https-proxy http://proxy.company.com:8080
  ``

Default are the environment variables *HTTP_PROXY* and *http_proxy* for **proxy** and 
 *HTTPS_PROXY*, *https_proxy*, *HTTP_PROXY* or *http_proxy* for **https-proxy**.


##### Java #####

If you want to run the Apache release audit tools ([Apache Rat™](https://creadur.apache.org/rat/) ) please perform the 
following steps:

* Navigate to 
``./olingo-odata4-js/grunt-config/custom-tasks/rat``
* Run ``npm install`` to install the node_modules required by the rat tool

then

* Proceed as described the filed ``readme.md``

##### C# #####

In order to run the service for testing you need the the "Microsoft Visual Studio SP1" with the extension 
 "NuGet package manager". 

Please install the following packages via "NuGet package manager":

* Microsoft.OData.Client -Version 6.5.0
* Microsoft.OData.Core -Version 6.5.0
* Microsoft.OData.Edm -Version 6.5.0
* Microsoft.OData.Service -Version 6.5.0
* Microsoft.OData.Spatial -Version 6.5.0
