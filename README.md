# aspects_php

Ansible Role to install and configure php, php-cli, apache mod_php, and the pecl libraries.


## Requirements

Set ```hash_behaviour=merge``` in your ansible.cfg file.

## Role Variables

See [defaults/main.yml](defaults/main.yml) for details.

* aspects_php_enabled
* aspects_php_apache_installed
* aspects_php_cli_installed
* aspects_php_config
* aspects_php_packages
* aspects_php_modules

### aspects_php_config
A dictionary/hash of php.ini settings that allows you to add, modify, and remove settings.

Use this pattern:

```yaml
aspects_php_config:
  <setting key>:
    enable: <True or False>
    remove: <True or False>
    target:
      <ansible_distribution>:
        <ansible_distribution_version or ansible_distribution_major_version>: "<target file path>"
    section: "<The ini section for the setting>"
    name: "<The setting key/name>"
    value: "<The setting value>"
```

For example, to add and/or modify:

```yaml
aspects_php_config:
  memory_limit_apache:
    enable: True
    target:
      Ubuntu:
        1404: "/etc/php5/apache2/99-aspects_php.ini"
        1604: "/etc/php/7.0/apache2/conf.d/99-aspects_php.ini"
      Debian:
        9: "/etc/php/7.0/apache2/conf.d/99-aspects_php.ini"
      CentOS:
        7: "/etc/php.d/99-aspects_php.ini"
    section: "PHP"
    name: "memory_limit"
    value: "256M"
  error_log_cli:
    enable: True
    target:
      Ubuntu:
        1404: "/etc/php5/cli/99-aspects_php.ini"
        1604: "/etc/php/7.0/cli/conf.d/99-aspects_php.ini"
      Debian:
        9: "/etc/php/7.0/cli/conf.d/99-aspects_php.ini"
      CentOS:
        7: "/etc/php.d/99-aspects_php.ini"
    section: "PHP"
    name: "error_log"
    value: "syslog"
```

#### Removing settings from files

To remove a setting from a file entirely, use something like this:
```yaml
aspects_php_config:
  memory_limit_apache:
    enable: False
    remove: True
    target:
      Ubuntu:
        1404: "/etc/php5/apache2/99-aspects_php.ini"
        1604: "/etc/php/7.0/apache2/conf.d/99-aspects_php.ini"
      Debian:
        9: "/etc/php/7.0/apache2/conf.d/99-aspects_php.ini"
      CentOS:
        7: "/etc/php.d/99-aspects_php.ini"
    section: "PHP"
    name: "memory_limit"
    value: "256M"
```

You need to set ```enable``` to ```False```, and ```remove``` to ```True```. Then you just make sure you are targetting the correct files.

#### Configuring Apache vs. CLI

> Note: I found that CentOS 7 does not differentiate between settings for Apache and settings for php-cli.

To configure Apache mod php and php-cli with different values for the same setting, simply add one item for apache, and one item for php-cli. Target the conf.d directory for Apache for the Apache item, and the conf.d directory for php-cli for the php-cli item.

#### Configuring multiple versions of PHP

If you have multiple versions of PHP installed, simply add an item to the ```aspects_php_config``` hash for each version targeting the appropriate conf.d directories.

### aspects_php_packages
A dictionary/hash of packages to install.

Use this pattern:

```yaml
aspects_php_packages:
  <package key>:
    state: "<present or latest>"
    <ansible_distribution>:
      <ansible_distribution_version or ansible_distribution_major_version>: "<package name>"
```
Set ```state``` to "default" if you wish to list a package but not install it.

Check the [tasks/aptInstallpackages.yml](aptInstallpackages.yml) or [tasks/yumInstallPackages.yml](yumInstallPackages.yml) files to find out what values are accepted for the ```ansible_distribution_*``` variables.


For example:

```yaml
aspects_php_packages:
  php:
    state: "present"
    Ubuntu:
      1604: "php"
      1404: "php5"
    Debian:
      9: "php"
    CentOS:
      7: "php"
  phpcurl:
    state: "present"
    Ubuntu:
      1404: "php5-curl"
      1604: "php-curl"
    Debian:
      9: "php-curl"
    CentOS:
      7: "php"
  phpgd:
    state: "present"
    Ubuntu:
      1404: "php5-gd"
      1604: "php-gd"
    Debian:
      9: "php-gd"
    CentOS:
      7: "php-gd"
```

## Enabling and Disabling mods with php5enmod and php5dismod

On Ubuntu 14.04 the default means of enabling and disabling php modules is to use the
php5enmod and php5dismod tools. These tools do more than just create links to ini files.
Thus there is a custom task file that uses these tools. These tasks only run on Ubuntu 14.04
and Debian 7.

To use this functionality, simply find the module name and the path of the link php5enmod creates,
then add it to the ```aspects_php_modules``` dictionary.

For example, the json module would look like this:

    aspects_php_modules:
      json:
        state: "enabled"
        name: "json"
        link: "/etc/php5/apache2/conf.d/20-json.ini"

## PECL Module configuration

PECL module tasks depend on Ansible knowing where the module .so file lives. This location varies greatly between distributions.

So, on Ubuntu 14.04 this would work:

    aspects_php_pecl_packages:
      apcu:
        state: "present"
        name: "channel://pecl.php.net/apcu-4.0.7"
        linecreated:
          Debian: "/usr/lib/php5/20121212/apcu.so"

But on Ubuntu 12.04, you would need:

    aspects_php_pecl_packages:
      apcu:
        state: "present"
        name: "channel://pecl.php.net/apcu-4.0.7"
        linecreated:
          Debian: "/usr/lib/php5/20090626/apcu.so"

To enable the modules, add the ```extension=mod.so``` line in an appropriate section via the ```aspects_php_config``` dictionary. Like:

    aspects_php_config:
      apcu:
        enable: True
        target:
          Debian:
            apache: "/etc/php5/apache2/php.ini"
          RedHat:
            apache: "/etc/php.ini"
        section: "apcu"
        name: "extension"
        value: "apcu.so"

If you have problems, make sure to test the PECL install manually. Also, use ```ansible-playbook -vvvv``` to find out exactly what is going on.

## Example Playbook

```yaml
- hosts:
  - hostname
  roles:
  - aspects_php
  vars:
    aspects_php_enabled: True
    aspects_php_apache_installed: True
    aspects_php_cli_installed: True
    aspects_php_packages:
      php:
        state: "present"
        Ubuntu:
          1604: "php"
          1404: "php5"
        Debian:
          9: "php"
        CentOS:
          7: "php"
      phpcurl:
        state: "present"
        Ubuntu:
          1404: "php5-curl"
          1604: "php-curl"
        Debian:
          9: "php-curl"
        CentOS:
          7: "php"
      phpgd:
        state: "present"
        Ubuntu:
          1404: "php5-gd"
          1604: "php-gd"
        Debian:
          9: "php-gd"
        CentOS:
          7: "php-gd"
    aspects_php_config:
      allow_url_fopen_apache:
        enable: True
        target:
          Ubuntu:
            1404: "/etc/php5/apache2/conf.d/99-aspects_php.ini"
            1604: "/etc/php/7.0/apache2/conf.d/99-aspects_php.ini"
          Debian:
            9: "/etc/php/7.0/apache2/conf.d/99-aspects_php.ini"
          CentOS:
            7: "/etc/php.d/99-aspects_php.ini"
        section: "PHP"
        name: "allow_url_fopen"
        value: "On"
      allow_url_fopen_cli:
        enable: True
        target:
          Ubuntu:
            1404: "/etc/php5/cli/conf.d/99-aspects_php.ini"
            1604: "/etc/php/7.0/cli/conf.d/99-aspects_php.ini"
          Debian:
            9: "/etc/php/7.0/cli/conf.d/99-aspects_php.ini"
        section: "PHP"
        name: "allow_url_fopen"
        value: "On"
      memory_limit_apache:
        enable: True
        target:
          Ubuntu:
            1404: "/etc/php5/apache2/conf.d/99-aspects_php.ini"
            1604: "/etc/php/7.0/apache2/conf.d/99-aspects_php.ini"
          Debian:
            9: "/etc/php/7.0/apache2/conf.d/99-aspects_php.ini"
          CentOS:
            7: "/etc/php.d/99-aspects_php.ini"
        section: "PHP"
        name: "memory_limit"
        value: "256M"
      error_log_cli:
        enable: True
        target:
          Ubuntu:
            1404: "/etc/php5/cli/conf.d/99-aspects_php.ini"
            1604: "/etc/php/7.0/cli/conf.d/99-aspects_php.ini"
          Debian:
            9: "/etc/php/7.0/cli/conf.d/99-aspects_php.ini"
        section: "PHP"
        name: "error_log"
        value: "syslog"
      memory_limit_cli:
        enable: True
        target:
          Ubuntu:
            1404: "/etc/php5/cli/conf.d/99-aspects_php.ini"
            1604: "/etc/php/7.0/cli/conf.d/99-aspects_php.ini"
          Debian:
            9: "/etc/php/7.0/cli/conf.d/99-aspects_php.ini"
          CentOS:
            7: "/etc/php.d/99-aspects_php.ini"
        section: "PHP"
        name: "memory_limit"
        value: "256M"
```

## License

MIT
