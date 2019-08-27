# aspects_php

Ansible Role to install and configure php, php-cli, and apache mod_php.


## Requirements

Set ```hash_behaviour=merge``` in your ansible.cfg file.

## Role Variables

### aspects_php_enabled

Default is ```False```.

Set to ```True``` to run tasks in this role.

Set to ```False``` to block all tasks in this role from running.

### aspects_php_config
A dictionary/hash of php.ini settings that allows you to add, modify, and remove settings.

You will need to set a specific target file per OS. If a host has more than one version of PHP installed, make sure you 
target the correct versions ini file or configuration directory.

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

For example, to add and/or modify the Apache mod_php memory limit:

```yaml
aspects_php_config:
  memory_limit_apache:
    enable: True
    target:
      Ubuntu:
        1604: "/etc/php/7.0/apache2/conf.d/99-aspects_php.ini"
        1804: "/etc/php/7.2/apache2/conf.d/99-aspects_php.ini"
      Debian:
        9: "/etc/php/7.0/apache2/conf.d/99-aspects_php.ini"
      CentOS:
        7: "/etc/php.d/99-aspects_php.ini"
      OracleLinux:
        7: "/etc/php.d/99-aspects_php.ini"
    section: "PHP"
    name: "memory_limit"
    value: "256M"
```

#### Removing settings from files

To remove a setting from a file entirely, you need to set `enable` to `False`, and `remove` to `True`. 
Then you just make sure you are targeting the correct files. 

For example, to remove the Apache mod_php memory limit setting from a custom config ini file:
```yaml
aspects_php_config:
  memory_limit_apache:
    enable: False
    remove: True
    target:
      Ubuntu:
        1604: "/etc/php/7.0/apache2/conf.d/99-aspects_php.ini"
        1804: "/etc/php/7.2/apache2/conf.d/99-aspects_php.ini"
      Debian:
        9: "/etc/php/7.0/apache2/conf.d/99-aspects_php.ini"
      CentOS:
        7: "/etc/php.d/99-aspects_php.ini"
      OracleLinux:
        7: "/etc/php.d/99-aspects_php.ini"
    section: "PHP"
    name: "memory_limit"
    value: "256M"
```

#### Configuring Apache vs. CLI

> Note: I found that CentOS 7 and Oracle Linux 7 do not differentiate between settings for Apache and settings for php-cli.

To configure Apache mod php and php-cli with different values for the same setting, simply add one item for apache, and 
one item for php-cli. Target the conf.d directory for Apache for the Apache item, and the conf.d directory for php-cli 
for the php-cli item.

#### Configuring multiple versions of PHP

If you have multiple versions of PHP installed, simply add an item to the `aspects_php_config` hash for each version 
targeting the appropriate conf.d directories.

### aspects_php_remove_package_managed_ini_files

When you install a php package from the various OS repositories, the packages often install default .ini configuration 
files. Debian/Ubuntu use the mods-available/mods-enabled symlink method, and CentOS just places a straight .ini file in 
the php.d directory. This presents a problem if you do not wish to use those files for custom configuration settings.

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

Once the files are removed, you can restart Apache and only your custom settings from `aspects_php_config` will apply.

## Enabling and Disabling PHP Modules

If you need to enable or disable a module that resides in your OS's package repositories, simply install or remove the 
associated package using [aspects_packages](https://github.com/LaneCommunityCollege/aspects_packages)  or whatever means you use to manage OS packages.

> Note: Depending on how your package manager deals with dependencies, you may need to add a second package for removal 
>to the list. I.E. Debian 9's php-ldap package has a dependency on the php7.0-ldap package. The ldap module does not get 
>disabled until you remove both packages.

Then make sure you have removed any custom configuration for that module from the `aspects_php_config` variable.

## Dependencies
### aspects_packages
* (optional) [aspects_packages](https://github.com/LaneCommunityCollege/aspects_packages) is used to manage system packages.

## Example Playbook

```yaml
- hosts:
  - aspects_php
  #  - aspectscentos7
  #  - aspectstrusty
  #  - aspectsxenial
  #  - aspectsstretch
  #  - aspectsoraclelinux7
  #  - aspectbionic
  roles:
  - aspects_php
  vars:
    aspects_packages_enabled: True
    aspects_php_enabled: True
    aspects_php_apache_installed: True
    aspects_php_cli_installed: True
    aspects_packages_packages:
      phpdev:
        state: "present"
        Ubuntu:
          1604: "php-dev"
          1804: "php-dev"
        Debian:
          9: "php-dev"
        CentOS:
          7: "php-devel"
        OracleLinux:
          7: "php54-php-devel"
      phppear:
        state: "present"
        Ubuntu:
          1604: "php-pear"
          1804: "php-pear"
        Debian:
          9: "php-pear"
        CentOS:
          7: "php-pear"
        OracleLinux:
          7: "php-pear"
      phpcurl:
        state: "present"
        Ubuntu:
          1604: "php-curl"
          1804: "php-curl"
        Debian:
          9: "php-curl"
      phpgd:
        state: "present"
        Ubuntu:
          1604: "php-gd"
          1804: "php-gd"
        Debian:
          9: "php-gd"
        CentOS:
          7: "php-gd"
        OracleLinux:
          7: "php-gd"
    aspects_php_config:
      allow_url_fopen_apache:
        enable: True
        target:
          Ubuntu:
            1604: "/etc/php/7.0/apache2/conf.d/99-aspects_php.ini"
            1804: "/etc/php/7.2/apache2/conf.d/99-aspects_php.ini"
          Debian:
            9: "/etc/php/7.0/apache2/conf.d/99-aspects_php.ini"
          CentOS:
            7: "/etc/php.d/99-aspects_php.ini"
          OracleLinux:
            7: "/etc/php.d/99-aspects_php.ini"
        section: "PHP"
        name: "allow_url_fopen"
        value: "On"
      allow_url_fopen_cli:
        enable: True
        target:
          Ubuntu:
            1604: "/etc/php/7.0/cli/conf.d/99-aspects_php.ini"
            1804: "/etc/php/7.2/cli/conf.d/99-aspects_php.ini"
          Debian:
            9: "/etc/php/7.0/cli/conf.d/99-aspects_php.ini"
        section: "PHP"
        name: "allow_url_fopen"
        value: "On"
      memory_limit_apache:
        enable: True
        target:
          Ubuntu:
            1604: "/etc/php/7.0/apache2/conf.d/99-aspects_php.ini"
            1804: "/etc/php/7.2/apache2/conf.d/99-aspects_php.ini"
          Debian:
            9: "/etc/php/7.0/apache2/conf.d/99-aspects_php.ini"
          CentOS:
            7: "/etc/php.d/99-aspects_php.ini"
          OracleLinux:
            7: "/etc/php.d/99-aspects_php.ini"
        section: "PHP"
        name: "memory_limit"
        value: "256M"
      error_log_cli:
        enable: True
        target:
          Ubuntu:
            1604: "/etc/php/7.0/cli/conf.d/99-aspects_php.ini"
            1804: "/etc/php/7.2/cli/conf.d/99-aspects_php.ini"
          Debian:
            9: "/etc/php/7.0/cli/conf.d/99-aspects_php.ini"
        section: "PHP"
        name: "error_log"
        value: "syslog"
      memory_limit_cli:
        enable: True
        target:
          Ubuntu:
            1604: "/etc/php/7.0/cli/conf.d/99-aspects_php.ini"
            1804: "/etc/php/7.2/cli/conf.d/99-aspects_php.ini"
          Debian:
            9: "/etc/php/7.0/cli/conf.d/99-aspects_php.ini"
          CentOS:
            7: "/etc/php.d/99-aspects_php.ini"
          OracleLinux:
            7: "/etc/php.d/99-aspects_php.ini"
        section: "PHP"
        name: "memory_limit"
        value: "256M"
```

## License

MIT
