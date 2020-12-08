## setSkyWalkingDynamicField error

```
Caused by: javassist.compiler.CompileError: setSkyWalkingDynamicField(java.lang.Object) not found in
	at javassist.compiler.TypeChecker.atMethodCallCore(TypeChecker.java:749)
	at javassist.compiler.TypeChecker.atCallExpr(TypeChecker.java:695)
	at javassist.compiler.JvstTypeChecker.atCallExpr(JvstTypeChecker.java:157)
	at javassist.compiler.ast.CallExpr.accept(CallExpr.java:46)
	at javassist.compiler.CodeGen.doTypeCheck(CodeGen.java:242)
	at javassist.compiler.CodeGen.atStmnt(CodeGen.java:330)
	at javassist.compiler.ast.Stmnt.accept(Stmnt.java:50)
	at javassist.compiler.CodeGen.atStmnt(CodeGen.java:351)
	at javassist.compiler.ast.Stmnt.accept(Stmnt.java:50)
	at javassist.compiler.CodeGen.atIfStmnt(CodeGen.java:398)
	at javassist.compiler.CodeGen.atStmnt(CodeGen.java:355)
	at javassist.compiler.ast.Stmnt.accept(Stmnt.java:50)
	at javassist.compiler.CodeGen.atStmnt(CodeGen.java:351)
	at javassist.compiler.ast.Stmnt.accept(Stmnt.java:50)
	at javassist.compiler.CodeGen.atMethodBody(CodeGen.java:292)
	at javassist.compiler.CodeGen.atMethodDecl(CodeGen.java:274)
	at javassist.compiler.ast.MethodDecl.accept(MethodDecl.java:44)
	at javassist.compiler.Javac.compileMethod(Javac.java:169)
	at javassist.compiler.Javac.compile(Javac.java:95)
	at javassist.CtNewMethod.make(CtNewMethod.java:74)
```

`https://github.com/apache/dubbo/issues/2830`
```
This is what we found

1.  SkyWalking agent will manipulate the dubbo service class, adding `setSkyWalkingDynamicField` and `getSkyWalkingDynamicField`.
2.  Dubbo tries to enhanced the classes too, but magically, dubbo core use the class skeleton from Class file, not SkyWalking agent enhanced. This trigger the conflict and exception.
```
### 添加解决冲突的jar包以及升级相关dubbo版本发现不好使
```
<dependency>
    <groupId>org.apache.skywalking</groupId>
    <artifactId>dubbo-conflict-patch</artifactId>
    <version>6.3.0</version>
</dependency>
```


### 将相关注解从facade层（带有dubbo的service注解）移到component层之后可用