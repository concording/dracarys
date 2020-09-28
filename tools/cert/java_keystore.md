# [Java: sun.security.provider.certpath.SunCertPathBuilderException: unable to find valid certification path to requested target]


## steps

1. export certfile from chrome  
   * First click on the certificate's icon in the trust hierarchy.
   * The certificate will be shown in the main part of the modal.
   * Click on the certificate's large icon in the main part of the modal. **Drag** the icon to your desktop. Chrome will then copy the certificate to your desktop.
2. import cert to jvm
   - `sudo keytool -list -v -keystore cacerts > java_cacerts.txt`
   - `sudo keytool -import -alias checkstyle -keystore  cacerts -file /Users/wangyinbin/Downloads/checkstyle.org.cer`
3. restart jvm
`ps -ef|grep java`




The `Java` process runs on demand as and when you want to run it. It's not a daemon. You need to stop the `Java` process manually (kill it) if it doesn't end gracefully.

[https://superuser.com/questions/97201/how-to-save-a-remote-server-ssl-certificate-locally-as-a-file](https://superuser.com/questions/97201/how-to-save-a-remote-server-ssl-certificate-locally-as-a-file)


[https://magicmonster.com/kb/prg/java/ssl/pkix_path_building_failed/](https://magicmonster.com/kb/prg/java/ssl/pkix_path_building_failed/)






