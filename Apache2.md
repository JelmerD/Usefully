# Apache2 specific stuff

## Default .conf file for Apache

- To use in /etc/apache2/sites-available
- Make sure to `a2enmod ssl` and `a2enmod macro`
- Add certificates by using `certbot certonly -d example.nl`, option 1 (apache server plugin). Make sure to comment out `use VHostRedirect 80 example.nl example.nl` whilst doing so, otherwise it won't work.
- Make sure to `systemctl reload apache2` for any changes to take effect
- use `apachectl -S` to see all current virtualhosts

/etc/sites-available/000-macros.conf
```xml
<Macro VHostRedirect $servname $redir>
<VirtualHost *:80>
        ServerName $servname
        Redirect permanent / https://$redir/
</VirtualHost>
</Macro>

<Macro VHostRewrite $servname $redir>
<VirtualHost *:443>
        ServerName $servname
        RewriteEngine on
        RewriteRule ^ https://$redir%{REQUEST_URI} [END,NE,R=permanent]
</virtualHost>
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

        ErrorLog ${APACHE_LOG_DIR}/$servname.error.log
        CustomLog ${APACHE_LOG_DIR}/$servname.access.log combined


        SSLCertificateFile /etc/letsencrypt/live/$servname/fullchain.pem
        SSLCertificateKeyFile /etc/letsencrypt/live/$servname/privkey.pem
        Include /etc/letsencrypt/options-ssl-apache.conf
</VirtualHost>
</Macro>
```

/etc/apache2/sites-available/example.nl.conf
```xml
<Macro VHostBlock $servname>
<IfModule mod_ssl.c>
use VHostSSL $servname $servname/www/webroot
use VHostSSL sub.$servname $servname/www/webroot
use VHostRewrite www.$servname $servname
</IfModule>

use VHostRedirect $servname $servame
use VHostRedirect www.$servname $servame
use VHostRedirect sub.$servname sub.$servame
</Macro>

use VHostBlock example.nl

UndefMacro VHostBlock
```
