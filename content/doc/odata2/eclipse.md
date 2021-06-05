Title:     Eclipse

# Eclipse IDE Support

---

[Eclipse](http://eclipse.org) (in version `Juno` and `Kepler`) is supported by the project.

With the Maven project files can be generated with:

`mvn eclipse:clean eclipse:eclipse`

As result each Maven module will get a consistent `.project`, `.classpath` and `.settings` file with which each module can be imported as existing project to Eclipse. 
