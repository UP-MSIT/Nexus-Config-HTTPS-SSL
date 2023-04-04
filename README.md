# Nexus-Config-HTTPS-SSL
## Nexus configuration set-up in AWS

[Download Nexus](https://download.sonatype.com/nexus/3/nexus-3.45.0-01-unix.tar.gz)

    cd /opt
    root@user:/opt# wget https://download.sonatype.com/nexus/3/nexus-3.45.0-01-unix.tar.gz
    
    root@user:/opt# tar -xzf nexus-3.45.0-01-unix.tar.gz
    root@user:/opt# mv nexus-3.45.0-01-unix.tar.gz nexus
    root@user:/opt# mv nexus-3.45.0-01-unix.tar.gz /var/tmp
    root@user:/opt# ls -l 
  

**Download Java jdk-8 and Installation steps:**

1.  **Install software source manager**
    
    ```java
    apt-get update
    apt-get install software-properties-common
    
    ```
    
2.  **Add mirror with openjdk-8-jdk**
    
    ```java
    apt-add-repository 'deb http://security.debian.org/debian-security stretch/updates main'
    apt-get update
    
    ```
    
3.  **Install openjdk 8**
    
    ```java
    apt-get install openjdk-8-jdk
    ```
4. **Check java version**
    ``` 
    java -version
    ```    
   
**Start nexus**
     
```/opt/nexus/bin/nexus/ start ```

**Nexus process**

``` ps -ef | grep nexus```

**Running port**

```netstat -ntap```


**Nexus and SonarQube logs**

```
tail -f /opt/sonatype-work/nexus3/log/nexus.log
tail -f /opt/sonarqube/logs/sonar.log 

```

**Install letsengrypt**

```apt install letsencrypt```

**Create directory called ssl in /opt**

``` 
mkdir ssl
cd ssl
```

**Display certication file path run command**

```
certbot --manual --preferred-challenges dns certonly -d *.yourDomainName.com
```

**After install letsencrypt copy following path**

```
/etc/letsencrypt/live/sithvothykiv.site/privkey.pem
/etc/letsencrypt/live/sithvothykiv.site/fullchain.pem
/etc/letsencrypt/live/sithvothykiv.site/cert.pem
```

**Combind fullchain and cert.pem together**

```
cat cert.pem >> new.pem 
cat fullchain.pem >> new.pem
```

**Generate keystore.p12 using new.pem**

```
openssl pkcs12 -export -in new.pem -inkey privkey.pem -certfile new.pem -out keystore.p12
keytool -importkeystore -srckeystore keystore.p12 -srcstoretype pkcs12 -destkeystore keystore.jks -deststoretype JKS
keytool -list -keystore keystore.jks
``` 
 
 **Edit jetty-https.xml file**
 ```
 root@user: cd /opt/nexus/etc/jetty
 root@user: vim jetty-https.xml
 ```  

**Go to /opt/sonatype-work/nexus3/etc Edit nexus.properties file**

```
root@user: cd /opt/sonatype-work/nexus3/etc
root@user: vim nexus.properties
```

**Install apache2 web server and create nexus.conf**

```
   apt install apache2 

   root@user: cd /etc/apache2/sites-available
   root@user: vim nexus.conf

<VirtualHost *:80>
        ServerName nexus.sithvothykiv.site
        ServerAdmin kivsethvuthy12@gmail.com

 RedirectMatch ^/(.*)$ https://nexus.sithvothykiv.site/$1

        LogLevel debug
        ErrorLog ${APACHE_LOG_DIR}/nexus-access.log
        CustomLog ${APACHE_LOG_DIR}/nexus-access.log combined
</VirtualHost>

<VirtualHost *:443>
        ServerAdmin kivsethvuthy12@gmail.com
        ServerName sithvothykiv.site

        SSLEngine On
        SSLCertificateFile /opt/ssl/cert.pem
        SSLCertificateKeyFile /opt/ssl/privkey.pem
        SSLCertificateChainFile /opt/ssl/fullchain.pem


 SSLProxyEngine on
        ProxyPreserveHost On
        ProxyRequests off
        SSLProxyProtocol +TLSv1.2
        ProxyPass             /        https://nexus.sithvothykiv.site:8443/
        ProxyPassReverse      /        https://nexus.sithvothykiv.site:8443/


        LogLevel debug
        ErrorLog ${APACHE_LOG_DIR}/nexus-error.log
        CustomLog ${APACHE_LOG_DIR}/nexus-access.log combined
</VirtualHost>

```
**Start service apache2**
```
    /etc/init.d/apache2 restart
    systemctl status apache2.service
```

## Push image to nexus

***Note: Stay in current project directory***

**Login to nexus repoistory**

```
financial-service-web % docker login https://nexus.sithvothykiv.site:8888
```

**Build docker image**

```
financial-service-web % docker build -t financial-service-web:v0.0.1 .
financial-service-web % docker images 
```

**Push docker image**


```
docker tag [OPTIONS] IMAGE[:TAG] [REGISTRYHOST/][USERNAME/]NAME[:TAG]

financial-service-web % docker tag 9d54c2cec032 nexus.sithvothykiv.site:8888/financial-service-web:v0.0.1 
financial-service-web % docker push nexus.sithvothykiv.site:8888/financial-service-web:v0.0.1

docker run [OPTIONS] IMAGE [COMMAND] [ARG...]

financial-service-web % docker run -itd --name financial-service-web 9d54c2cec032
```


