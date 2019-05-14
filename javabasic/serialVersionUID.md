
`private static final long serialVersionUID = -1672970955045193907L;`

SerialVersionUID is an ID which is stamped on object when it get serialized usually hashcode of object, you can use tool serialver to see serialVersionUID of a serialized object . SerialVersionUID is used for version control of object. you can specify serialVersionUID in your class file also. Consequence of not specifying serialVersionUID is that when you add or modify any field in class then already serialized class will not be able to recover because serialVersionUID generated for new class and for old serialized object will be different. Java serialization process relies on correct serialVersionUID for recovering state of serialized object and throws java.io.InvalidClassException in case of serialVersionUID mismatch.

Please don’t simply copy and paste from another website; that may be breach of copyright. That quote is confusing at the best.

An SUID is not a hash of the object, but a hash of its originating class. If the class is updated, for example with different fields, the SUID can change. You have four (at least) possible courses of action:-

1: Leave out the SUID. This tells the runtime that there are no differences between versions of classes when serialising and deserialising.2: Always write a default SUID, which looks like the heading of this thread. That tells the JVM that all versions with this SUID count as the same version.3: Copy an SUID from a previous version of the class. That tells the runtime that this version and the version the SUID came from are to be treated as the same version.4: Use a generated SUID for each version of the class. If the SUID is different in a new version of the class, that tells the runtime that the two versions are different and serialised instances of the old class cannot be deserialised as instances of the new class.
