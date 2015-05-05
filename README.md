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

Example Playbook
-------------------------

Including an example of how to use your role (for instance, with variables passed in as parameters) is always nice for users too:

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
