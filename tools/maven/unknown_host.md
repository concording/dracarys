
```
[ERROR] Failed to execute goal on project loanrisk-service: Could not resolve dependencies for project 
Failed to collect dependencies at ***** : Failed to read artifact descriptor for ***** : Could not transfer artifact ******:pom:2.0.1 from/to central (https://repo.maven.apache.org/maven2): repo.maven.apache.org: unknown error: Unknown host repo.maven.apache.org: unknown error -> [Help 1]
```

If nothing wrong with your connection, try the following:

Remove all the .lastUpdate files under .m2 repository.
In Eclipse, right click on your project and select Maven->Update Project.
Choose Force Update of Snapshots/Releases
