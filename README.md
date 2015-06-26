aspects_php
=========
Ansible Role to install and configure php, php-cli, apache mod_php, and the pecl libraries.


Requirements
------------

Set ```hash_behaviour=merge``` in your ansible.cfg file.

Role Variables
--------------

See defaults/main.yml for details.

* aspects_php_enabled
* aspects_php_apache_installed
* aspects_php_cli_installed
* aspects_php_config
* aspects_php_packages

Enabling and Disabling mods with phpenmod and phpdismod
-------------------------------------------------------

On Ubuntu 14.04 the default means of enabling and disabling php modules is to use the
phpenmod and phpdismod tools. These tools do more than just create links to ini files. 
Thus there is a custom task file that uses these tools. These tasks only run on Ubuntu 14.04 
and Debian 7.

To use this functionality, simply find the module name and the path of the link phpenmod creates,
then add it to the ```aspects_php_modules``` dictionary.

For example, the json module would look like this:

    aspects_php_modules: 
      json:
        state: "enabled"
        name: "json"
        link: "/etc/php5/apache2/conf.d/20-json.ini"

Example Playbook
-------------------------


    - hosts:
      - hostname
      roles:
      - aspects_php
      vars:
        aspects_php_enabled: True
        aspects_php_apache_installed: True
        aspects_php_cli_installed: True
        aspects_php_packages:
          libapache2modphp5:
            state: "latest"
            Debian: "libapache2-mod-php5"
            RedHat: "httpd"
        aspects_php_config:
          outputbuffering:
            enable: False
            target:
              Debian:
                apache: "/etc/php5/apache2/php.ini"
              RedHat:
                apache: "/etc/php.ini"
            section: "PHP"
            name: "output_buffering"
            value: "4096"

License
-------

MIT
