# Install [InvoiceNinja](https://app.invoiceninja.com/invoice_now?rc=GCYXTIBU) on [Uberspace](https://uberspace.de/)

![](https://www.invoiceninja.com/wp-content/themes/invoice-ninja/images/laptop.jpg)
<sup>Image by InvoiceNinja</sup>

## 1. Checkout the code

Say you want to host it under: `invoice.domain.com`:

```sh
cd $HOME/$USER
git clone https://github.com/invoiceninja/invoiceninja.git invoice.domain.com
```

### Add the domain to Uberspace

```sh
uberspace-add-domain -d invoice.domain.com -w
```

### Setup a HTTPS certificate

See here: [Let's-Encrypt-Zertifikate](https://wiki.uberspace.de/webserver:https?s[]=encrypt#let_s-encrypt-zertifikate).


## 2. Fix the `.htaccess`-files

Within `<IfModule mod_rewrite.c>` in `.htaccess` add:

```sh
  # Redirect HTTP to HTTPS:
  RewriteCond %{ENV:HTTPS} !=on
  RewriteRule ^(.*) https://%{HTTP_HOST}%{REQUEST_URI} [L,R=301]

  # Make InvoiceNinja work:
  RewriteBase /
  RewriteRule ^(.*)$ public/$1 [L]
```

Within `public/.htaccess` add:
```sh
# Make InvoiceNinja work with subdomains:
# If you want to access InvoiceNinja via a subdomain and
# this rule is missing the browser will see a `500 internal server error`.
RewriteBase /
```

## 3. Update PHP

```sh
echo "PHPVERSION=7.0" > ~/etc/phpversion
killall php-cgi
```

Make sure that your php-cli is set to verion 7.0 with `php -v`.
If not try to renew your ssh session.


## 4. Install composer

```sh
# https://getcomposer.org/download/
php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"
# If you see "Installer corrupt" the composer-setup.php file may have been updated.
# Then make sure composer-setup.php is not corrupt and ignore this line.
php -r "if (hash_file('SHA384', 'composer-setup.php') === 'e115a8dc7871f15d853148a7fbac7da27d6c0030b848d9b3dc09e2a0388afed865e6a3d6b3c0fad45c48e2b5fc1196ae') { echo 'Installer verified'; } else { echo 'Installer corrupt'; unlink('composer-setup.php'); } echo PHP_EOL;"
mkdir ~/.composer
php composer-setup.php --install-dir=${HOME}/.composer
php -r "unlink('composer-setup.php');"
```

Create a shellscript for composer in `~/bin/composer`:
```
cat > ${HOME}/bin/composer << __EOF__
sh
#/bin/bash
php ~/.composer/composer.phar "$@"
__EOF__
```

Then make it executable with `chmod +x ~/bin/composer`.


## 5. Install dependencies

Next switch to your to the previous created github clone (e.g. `cd ${HOME}/${USER}/invoice.domain.com`).
```sh
composer install
```

## 6. Set up crontab

If you set up crontab then InvoiceNinja will be able to automatically send out recurring invoices. Otherwise this step is not necessary.

```sh
0 8 * * * /usr/local/bin/php /var/www/virtual/<<USER_NAME>>/invoice.domain.com/artisan ninja:send-invoices
0 8 * * * /usr/local/bin/php /var/www/virtual/<<USER_NAME>>/invoice.domain.com/artisan ninja:send-reminders
```


## 7. Create a MySQL database

See here: [MySQL](https://wiki.uberspace.de/database:mysql?s[]=mysql).


## 8. Configure InvoiceNinja

In `config/app.php` set `APP_URL` to `https://invoice.domain.com` and `APP_KEY` to a [32 character long random string](https://www.random.org/strings/?num=2&len=16&digits=on&upperalpha=on&loweralpha=on&unique=on&format=html&rnd=new).

Perfom the final steps via the frontend of InvoiceNinja in this example that would be on `https://invoice.domain.com`.

--

If something goes wrong check the [troubleshooting section](https://www.invoiceninja.com/self-host/) in the official guide.

--
[![Flattr this](https://button.flattr.com/flattr-badge-large.png)](https://flattr.com/submit/auto?fid=2pkq07&url=https%3A%2F%2Fgithub.com%2Fpguth%2Finstall-invoiceninja) :rocket:
