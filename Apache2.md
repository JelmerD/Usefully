# Apache2 specific stuff

## Default .conf file for Apache

- To use in /etc/apache2/sites-available
- Make sure to `a2enmod ssl` and `a2enmod macro`
- Add certificates by using `certbot certonly -d example.nl`, option 1 (apache server plugin). Make sure to comment out `use VHostRedirect 80 example.nl example.nl` whilst doing so, otherwise it won't work.
- use `apachectl -S` to see all current virtualhosts

```xml
<Macro VHostRedirect $port $servname $redir>
<VirtualHost *:$port>
        ServerName $servname
        Redirect permanent / https://$redir/
</VirtualHost>
</Macro>

<Macro VHostSSL $servname $dir>
<VirtualHost *:443>
        ServerName $servname
        ServerAdmin webmaster@example.nl
        DocumentRoot /var/www/$dir

        <Directory /var/www/$dir>
                Options Indexes FollowSymLinks MultiViews
                AllowOverride all
                Order allow,deny
                allow from all
        </Directory>

        # Possible values include: debug, info, notice, warn, error, crit,
        # alert, emerg.
        LogLevel warn

        ErrorLog ${APACHE_LOG_DIR}/www.$servname.error.log
        CustomLog ${APACHE_LOG_DIR}/www.$servname.access.log combined


        SSLCertificateFile /etc/letsencrypt/live/$servname/fullchain.pem
        SSLCertificateKeyFile /etc/letsencrypt/live/$servname/privkey.pem
        Include /etc/letsencrypt/options-ssl-apache.conf
</VirtualHost>
</Macro>

<IfModule mod_ssl.c>
use VHostSSL example.nl example.nl/www/webroot
use VHostSSL sub.example.nl example.nl/www/webroot
use VHostRedirect 443 www.example.nl example.nl
</IfModule>

use VHostRedirect 80 example.nl example.nl
use VHostRedirect 80 www.example.nl example.nl
use VHostRedirect 80 sub.example.nl sub.example.nl

UndefMacro VHostRedirect
UndefMacro VHostSSL
```
