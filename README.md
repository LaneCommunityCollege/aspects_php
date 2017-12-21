# aspects_php

Ansible Role to install and configure php, php-cli, apache mod_php, and the pecl libraries.


## Requirements

Set ```hash_behaviour=merge``` in your ansible.cfg file.

## Role Variables

### aspects_php_enabled

Default is ```False```.

Set to ```True``` to run tasks in this role.

Set to ```False``` to block all tasks in this role from running.

### aspects_php_apache_installed
### aspects_php_cli_installed

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

You need to set ```enable``` to ```False```, and ```remove``` to ```True```. Then you just make sure you are targeting the correct files.

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

### aspects_php_remove_package_managed_ini_files

When you install a php package from the various OS repositories, the packages often install default .ini configuration files. Debian/Ubuntu use the mods-available/mods-enabled symlink method, and CentOS just places a straight .ini file in the php.d directory. This presents a problem if you do not wish to use those files for custom configuration settings.

That is what this dictionary/hash is for. It is a list of files/symlinks that you want removed from your system.

```yaml
aspects_php_remove_package_managed_ini_files:
  <some key that makes sense>: "<path to file or symlink you want removed>"
```

For example:

```yaml
aspects_php_remove_package_managed_ini_files:
  modgdcentos: "/etc/php.d/gd.ini"
  modgddebian: "/etc/php/7.0/apache2/conf.d/20-gd.ini"
```

Once the files are removed, you can restart Apache and only your custom settings from ```aspects_php_config``` will apply.

### aspects_php_use_pecl
Default is ```False```.

Set to ```True``` to run PECL/PEAR installation or removal tasks.

Set to ```False``` to skip PECL/PEAR installation and removal tasks.

### aspects_php_pecl_packages

<b>Broken.</b>

#### Requirements

```aspects_php_use_pecl``` set to ```True```.

The php-dev and php-pear system packages must be installed for PECL/PEAR to work. Add the following to your ```aspects_php_packages``` dictionary:

```yaml
  phpdev:
    state: "present"
    Ubuntu:
      1604: "php-dev"
      1404: "php5-dev"
    Debian:
      9: "php-dev"
    CentOS:
      7: "php-devel"
  phppear:
    state: "present"
    Ubuntu:
      1404: "php-pear"
      1604: "php-pear"
    Debian:
      9: "php-pear"
    CentOS:
      7: "php-pear"
```

Use the pattern:

```yaml
aspects_php_pecl_packages:
  <package key>:
    state: "<present or latest>"
    name: "pecl/<packagename>"
```

For example:

```yaml
aspects_php_pecl_packages:
  uploadprogress:
    state: "present"
    name: "pecl/uploadprogress"
```

#### Enabling the installed module

Once the package is installed, you will need to enable the module using the ```aspects_php_config``` dictionary.

For example:

```yaml
aspects_php_config:
  uploadprogress:
    enable: True
    target:
      Ubuntu:
        1404: "/etc/php5/apache2/99-aspects_php.ini"
        1604: "/etc/php/7.0/apache2/conf.d/99-aspects_php.ini"
      Debian:
        9: "/etc/php/7.0/apache2/conf.d/99-aspects_php.ini"
      CentOS:
        7: "/etc/php.d/99-aspects_php.ini"
    section: "uploadprogress"
    name: "uploadprogress"
    value: "uploadprogress.so"
```

> Note: aspects_php uses the (http://docs.ansible.com/ansible/latest/pear_module.html)[Ansible pear] module to install PECL and PEAR packages. If you run into issues when installing a PECL/PEAR package, I suggest experimenting with the Ansible module directly on a vagrant vm. For example, I was using just "uploadprogress" and it was not working. Once I checked the docs and then tried "pecl/uploadprogress", it did work.

## Enabling and Disabling PHP Modules

If you need to enable or disable a module that resides in your OS's package repositories, simply install or remove the associated package using the ```aspects_php_packages``` variable.

> Note: Depending on how your package manager deals with dependencies, you may need to add a second package for removal to the list. I.E. Debian 9's php-ldap package has a dependency on the php7.0-ldap package. The ldap module does not get disabled until you remove both packages.

Then make sure you have removed any custom configuration for that module from the ```aspects_php_config``` variable.

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
      phpdev:
        state: "present"
        Ubuntu:
          1604: "php-dev"
          1404: "php5-dev"
        Debian:
          9: "php-dev"
        CentOS:
          7: "php-devel"
      phppear:
        state: "present"
        Ubuntu:
          1404: "php-pear"
          1604: "php-pear"
        Debian:
          9: "php-pear"
        CentOS:
          7: "php-pear"
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
    aspects_php_pecl_packages:
      uploadprogress:
        state: "present"
        name: "pecl/uploadprogress"
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
      uploadprogress:
        enable: True
        target:
          Ubuntu:
            1404: "/etc/php5/apache2/99-aspects_php.ini"
            1604: "/etc/php/7.0/apache2/conf.d/99-aspects_php.ini"
          Debian:
            9: "/etc/php/7.0/apache2/conf.d/99-aspects_php.ini"
          CentOS:
            7: "/etc/php.d/99-aspects_php.ini"
        section: "uploadprogress"
        name: "uploadprogress"
        value: "uploadprogress.so"
```

## License

MIT
