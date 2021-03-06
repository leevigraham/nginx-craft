# nginx-craft

An Nginx virtual host configuration for Craft CMS that implements a number of best-practices.

## Overview

### What it handles

The Nginx-Craft configuration handles:

* Redirecting from HTTP to HTTPS
* Canonical domain rewrites from www.SOMEDOMAIN.com to SOMEDOMAIN.com
* 301 Redirect URLs with trailing /'s as per https://webmasters.googleblog.com/2010/04/to-slash-or-not-to-slash.html
* Setting `PATH_INFO` properly via php-fpm -> PHP
* Setting `HTTP_HOST` to mitigate [HTTP_HOST Security Issues](https://expressionengine.com/blog/http-host-and-server-name-security-issues)
* "Far-future" Expires headers
* Enable serving of static gzip files via [gzip_static](http://nginx.org/en/docs/http/ngx_http_gzip_static_module.html)
* Adding XSS and other security headers
* Gzip compression
* Filename-based cache busting for static resources
* IPv4 and IPv6 support
* http2 support
* Reasonable SSL cipher suites and TLS protocols
* Localized sites
* Server-side includes
* Optionally includes [Dotenvy](https://github.com/nystudio107/dotenvy) generated `.env` files

### Assumptions made

The following are assumptions made in this configuration:

* The site is https
* The SSL certificate is from LetsEncrypt.com
* The canonical domain is SOMEDOMAIN.com (no www.)
* Nginx is version 1.9.5 or later (and thus supports http2)
* Paths are standard Ubuntu, change as needed
* You're using php7.1 via php-fpm
* You have `'omitScriptNameInUrls' => true,` in your `craft/general.php`

If any of these assumptions are invalid, make the appropriate changes.

**Note**: We disable TLSv1.0 because it is insecure, but IE 8, 9 & 10 need to have support for TLSv1.1 [manually enabled or they will not be able to connect](https://answers.microsoft.com/en-us/ie/forum/ie10-windows_other/disabling-tlsv10-breaks-compatibility-with-ie-9/80e77823-0f0c-49a8-b525-15ce6d7a570d?auth=1).

### What's included

This Nginx configuration comes in two parts:

* `sites-available/somedomain.com.conf` - an Nginx virtual host configuration file tailored for Craft CMS; it will require some minor customization for your domain
* `nginx-partials` - some Nginx configuration partials used by all of the virtual hosts, logically segregated.  These don't need to be changed, but can be selectively disabled by changing the suffix to `.off` (or anything other than `.conf`)

## Using Nginx-Craft

1. Obtain an SSL certificate for your domain via [LetsEncrypt.com](https://letsencrypt.org/) (or via other certificate authorities).  LetsEncrypt.com is free, and it's automated.  You will need a basic server up and running that responds to port 80 to do this, [LetsEnecrypt/Nginx tutorial](https://www.digitalocean.com/community/tutorials/how-to-secure-nginx-with-let-s-encrypt-on-ubuntu-16-04)
2. Create a `dhparam.pem` via `sudo openssl dhparam -out /etc/nginx/dhparams.pem 2048`
3. Download your Issuer certificate via `mkdir /etc/nginx/ssl; sudo wget -O /etc/nginx/ssl/lets-encrypt-x3-cross-signed.pem "https://letsencrypt.org/certs/lets-encrypt-x3-cross-signed.pem"`
4. Upload the entire `nginx-partials` folder to `/etc/nginx/`
5. Rename the `somedomain.com.conf` file to `yourdomain.com.conf`
6. Do a search & replace in `yourdomain.com.conf` to change `SOMEDOMAIN` -> `yourdomain`
7. Tweak any paths that may need changing on your server
8. Change the `fastcgi_pass unix:/var/run/php/php7.1-fpm.sock;` line to reflect whatever version of PHP you're running
9. Restart nginx via `sudo nginx -s reload`

If you're using [Forge](https://forge.laravel.com/), it takes care of a number of these things for you, but still needs tuning. 

The same applies for CloudWays, ServerPilot, Homestead, MAMP, etc.

A [Forge Template](https://forge.laravel.com/docs/1.0/servers/nginx-templates.html) is provided in `forge-templates/NginxTemplate.conf` that you can use to [automate setting up](https://blog.laravel.com/forge-nginx-templates) your Forge servers.

For this to work, you must clone the repo into `/home/forge` via:
```
git clone https://github.com/nystudio107/nginx-craft.git /home/forge
```

For further information on TLS optimization, see the [How to properly configure your nginx for TLS](https://medium.com/@mvuksano/how-to-properly-configure-your-nginx-for-tls-564651438fe0) article.

## Forge & opcache

**N.B.:** Forge now has `opcache` functionality baked-in, you can enable it via the Server settings, so this information is largely deprecated.

If you're using Forge, understand that `opcache` is off by default. To enable it, go to your server in Forge, click on *Edit Files* and choose *Edit PHP FPM Configuration* and search on `opcache`. Here are the defaults I use; tweak them to suit your needs:

    [opcache]
    ; Determines if Zend OPCache is enabled
    opcache.enable=1

    ; Determines if Zend OPCache is enabled for the CLI version of PHP
    ;opcache.enable_cli=0

    ; The OPcache shared memory storage size.
    opcache.memory_consumption=256

    ; The amount of memory for interned strings in Mbytes.
    opcache.interned_strings_buffer=16

    ; The maximum number of keys (scripts) in the OPcache hash table.
    ; Only numbers between 200 and 100000 are allowed.
    opcache.max_accelerated_files=8000

    ; If disabled, all PHPDoc comments are dropped from the code to reduce the
    ; size of the optimized code.
    opcache.save_comments=0

More about tweaking `opcache` can be found in the [Fine-Tune Your Opcache Configuration to Avoid Caching Suprises](https://tideways.io/profiler/blog/fine-tune-your-opcache-configuration-to-avoid-caching-suprises) article. The [Best Zend OpCache Settings/Tuning/Config](https://www.scalingphpbook.com/blog/2014/02/14/best-zend-opcache-settings.html) article is very useful as well.

## Local Development

While all of the configuration in the `somedomain.com.conf` will work fine in local development as well, some people might want a simpler setup for local development.

There is a `basic_localdev.com.conf` that you can use for a basic Nginx configuration that will work with Craft without any of the bells, whistles, or optimizations found in the `somedomain.com.conf`.

While this is suitable for getting up and running quickly for local development, do not use it in production. There are a number of performance optimizations missing from it.

Brought to you by [nystudio107](https://nystudio107.com/)
