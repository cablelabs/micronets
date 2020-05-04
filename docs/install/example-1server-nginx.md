## Example nginx configuration file with all Micronets services on one host

### for NCCoE Build 3

The following example nginx configuration file can be saved
in the `/etc/nginx/sites-available` directory. e.g. 
`/etc/nginx/sites-available/my-server.org`:

```
server {
    server_name my-server.org; # managed by Certbot

    location /micronets/mso-portal/ {
        proxy_pass http://127.0.0.1:3210/;
    }

    location /micronets/mud/ {
        proxy_pass      http://localhost:3082/registry;
    }

    location /registry/devices {
        proxy_pass      http://localhost:3082/vendors/;
    }

    location /micronets/mud-manager/ {
        proxy_pass      http://localhost:8888/;
    }

    include /etc/nginx/micronets-subscriber-forwards/*.conf;

    listen [::]:443 ssl ipv6only=on; # managed by Certbot
    listen 443 ssl; # managed by Certbot
    ssl_certificate /etc/letsencrypt/live/my-server.org/fullchain.pem; # managed by Certbot
    ssl_certificate_key /etc/letsencrypt/live/my-server.org/privkey.pem; # managed by Certbot
    include /etc/letsencrypt/options-ssl-nginx.conf; # managed by Certbot
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; # managed by Certbot
}
```

to enable the server, a symlink to the file in `/etc/nginx/sites-enabled` 
should be setup using:

```
sudo ln -sv /etc/nginx/sites-available/my-server.org /etc/nginx/sites-enabled/
```

nginx config files can be reloaded using:

```
sudo nginx -s reload
```

If the server isn't configured with a web (https) certificate, one can easily
be obtained using the Let's Encrypt Certbot. For example the server entry above
was setup using:

```
wget https://dl.eff.org/certbot-auto
sudo install -v -o root -m 0755 -D -t /usr/local/bin/ certbot-auto
sudo certbot-auto --nginx
```

The certbot-auto script will prompt for your server's registered DNS name, 
verify ownership, generate a cert, and configure your `server` entry to use
the certificate and associated private key for https connectios. 

Here's an example of a certbot provisioning session:

```
$ sudo certbot-auto --nginx
Saving debug log to /var/log/letsencrypt/letsencrypt.log
Plugins selected: Authenticator nginx, Installer nginx
Enter email address (used for urgent renewal and security notices) (Enter 'c' to
cancel): c.pratt@cablelabs.com

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Please read the Terms of Service at
https://letsencrypt.org/documents/LE-SA-v1.2-November-15-2017.pdf. You must
agree in order to register with the ACME server at
https://acme-v02.api.letsencrypt.org/directory
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
(A)gree/(C)ancel: a

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Would you be willing to share your email address with the Electronic Frontier
Foundation, a founding partner of the Let's Encrypt project and the non-profit
organization that develops Certbot? We'd like to send you email about our work
encrypting the web, EFF news, campaigns, and ways to support digital freedom.
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
(Y)es/(N)o: n
No names were found in your configuration files. Please enter in your domain
name(s) (comma and/or space separated)  (Enter 'c' to cancel): my-server.org
Obtaining a new certificate
Performing the following challenges:
http-01 challenge for my-server.org
Waiting for verification...
Cleaning up challenges
Deploying Certificate to VirtualHost /etc/nginx/sites-enabled/default
o
Please choose whether or not to redirect HTTP traffic to HTTPS, removing HTTP access.
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
1: No redirect - Make no further changes to the webserver configuration.
2: Redirect - Make all requests redirect to secure HTTPS access. Choose this for
new sites, or if you're confident your site works on HTTPS. You can undo this
change by editing your web server's configuration.
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Select the appropriate number [1-2] then [enter] (press 'c' to cancel):2
Redirecting all traffic on port 80 to ssl in /etc/nginx/sites-enabled/default

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Congratulations! You have successfully enabled https://my-server.org

You should test your configuration at:
https://www.ssllabs.com/ssltest/analyze.html?d=my-server.org
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -

IMPORTANT NOTES:
 - Congratulations! Your certificate and chain have been saved at:
   /etc/letsencrypt/live/my-server.org/fullchain.pem
   Your key file has been saved at:
   /etc/letsencrypt/live/my-server.org/privkey.pem
   Your cert will expire on 2020-07-26. To obtain a new or tweaked
   version of this certificate in the future, simply run certbot-auto
   again with the "certonly" option. To non-interactively renew *all*
   of your certificates, run "certbot-auto renew"
 - Your account credentials have been saved in your Certbot
   configuration directory at /etc/letsencrypt. You should make a
   secure backup of this folder now. This configuration directory will
   also contain certificates and private keys obtained by Certbot so
   making regular backups of this folder is ideal.
 - If you like Certbot, please consider supporting our work by:

   Donating to ISRG / Let's Encrypt:   https://letsencrypt.org/donate
   Donating to EFF:                    https://eff.org/donate-le

```