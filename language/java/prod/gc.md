### # JceSecurity导致JVM内存溢出、CPU过高问题排查


![CMS Old Gen](http://47.75.156.128/images/blog/gc/cms_old_gen.png)

![CMS Old Gen](http://47.75.156.128/images/blog/gc/cms_old_gen.png)

JceSecurity这个类就占用大部分内存，点击Dominator Tree进行分析，如下图：
![](http://47.75.156.128/images/blog/gc/jmap.png)



