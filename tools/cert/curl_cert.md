## Certificate Verification

libcurl performs peer SSL certificate verification by default. This is done by using a CA certificate store that the SSL library can use to make sure the peer's server certificate is valid.

If you communicate with HTTPS, FTPS or other TLS-using servers using certificates that are signed by CAs present in the store, you can be sure that the remote server really is the one it claims to be.

If the remote server uses a self-signed certificate, if you don't install a CA cert store, if the server uses a certificate signed by a CA that isn't included in the store you use or if the remote host is an impostor impersonating your favorite site, and you want to transfer files from this server, do one of the following:

1.  Tell libcurl to _not_ verify the peer. With libcurl you disable this with `curl_easy_setopt(curl, CURLOPT_SSL_VERIFYPEER, FALSE);`

    With the curl command line tool, you disable this with -k/--insecure.

2.  Get a CA certificate that can verify the remote server and use the proper option to point out this CA cert for verification when connecting. For libcurl hackers: `curl_easy_setopt(curl, CURLOPT_CAPATH, capath);`

    With the curl command line tool: --cacert [file]

3.  Add the CA cert for your server to the existing default CA certificate store. The default CA certificate store can changed at compile time with the following configure options:

    --with-ca-bundle=FILE: use the specified file as CA certificate store. CA certificates need to be concatenated in PEM format into this file.

    --with-ca-path=PATH: use the specified path as CA certificate store. CA certificates need to be stored as individual PEM files in this directory. You may need to run c_rehash after adding files there.

    If neither of the two options is specified, configure will try to auto-detect a setting. It's also possible to explicitly not hardcode any default store but rely on the built in default the crypto library may provide instead. You can achieve that by passing both --without-ca-bundle and --without-ca-path to the configure script.

    If you use Internet Explorer, this is one way to get extract the CA cert for a particular server:

    *   View the certificate by double-clicking the padlock
    *   Find out where the CA certificate is kept (Certificate> Authority Information Access>URL)
    *   Get a copy of the crt file using curl
    *   Convert it from crt to PEM using the openssl tool: openssl x509 -inform DES -in yourdownloaded.crt -out outcert.pem -text
    *   Add the 'outcert.pem' to the CA certificate store or use it stand-alone as described below.

    If you use the 'openssl' tool, this is one way to get extract the CA cert for a particular server:

    *   `openssl s_client -showcerts -servername server -connect server:443 > cacert.pem`
    *   type "quit", followed by the "ENTER" key
    *   The certificate will have "BEGIN CERTIFICATE" and "END CERTIFICATE" markers.
    *   If you want to see the data in the certificate, you can do: "openssl x509 -inform PEM -in certfile -text -out certdata" where certfile is the cert you extracted from logfile. Look in certdata.
    *   If you want to trust the certificate, you can add it to your CA certificate store or use it stand-alone as described. Just remember that the security is no better than the way you obtained the certificate.
4.  If you're using the curl command line tool, you can specify your own CA cert path by setting the environment variable `CURL_CA_BUNDLE` to the path of your choice.

    If you're using the curl command line tool on Windows, curl will search for a CA cert file named "curl-ca-bundle.crt" in these directories and in this order:

    1.  application's directory
    2.  current working directory
    3.  Windows System directory (e.g. C:\windows\system32)
    4.  Windows Directory (e.g. C:\windows)
    5.  all directories along %PATH%
5.  Get a better/different/newer CA cert bundle! One option is to extract the one a recent Firefox browser uses by running 'make ca-bundle' in the curl build tree root, or possibly download a version that was generated this way for you: [CA Extract](https://curl.haxx.se/docs/caextract.html)

Neglecting to use one of the above methods when dealing with a server using a certificate that isn't signed by one of the certificates in the installed CA certificate store, will cause SSL to report an error ("certificate verify failed") during the handshake and SSL will then refuse further communication with that server.