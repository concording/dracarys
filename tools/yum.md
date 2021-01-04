
```
yum install https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm

vi /etc/yum/pluginconf.d/subscription-manager.conf
enable=0

yum install zbar
```


```
updates                                                                                                                                                                                                                | 2.9 kB  00:00:00
  File "/usr/libexec/urlgrabber-ext-down", line 28
    except OSError, e:
                  ^
SyntaxError: invalid syntax
  File "/usr/libexec/urlgrabber-ext-down", line 28
    except OSError, e:
                  ^
SyntaxError: invalid syntax
  File "/usr/libexec/urlgrabber-ext-down", line 28
    except OSError, e:
                  ^
SyntaxError: invalid syntax
  File "/usr/libexec/urlgrabber-ext-down", line 28
    except OSError, e:
                  ^
SyntaxError: invalid syntax
  File "/usr/libexec/urlgrabber-ext-down", line 28
    except OSError, e:
                  ^
SyntaxError: invalid syntax


Exiting on user cancel
```


```
sudo vi /usr/libexec/urlgrabber*

change
#! /usr/bin/python to #! /usr/bin/python2
```
