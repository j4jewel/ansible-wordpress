# ansible-wordpress
This is an Ansible playbook to install wordpress on amazon linux. This will also install lamp on the ec2 server.

## Variables
##### Change it according to your need.
---
    domain: www.example.com
    mysql_root_password: mysqlroot123
    mysql_user: wpuser
    mysql_password: wpuser123
    mysql_database: wordpress 
---
Playbook
---

```sh
---
- name: 'Creating VirtualHost Configuration File'
  hosts: amazon
  become: yes
  vars:
    domain: www.example.com
    mysql_root_password: mysqlroot123
    mysql_user: wpuser
    mysql_password: wpuser123
    mysql_database: wordpress 
        
  tasks:
    - name: 'Installing Apache'
      yum: name=httpd,php,php-mysql state=present
        
    - name: 'Creating Virtualhost Configuration File'
      template:
        src: virtualhost.j2
        dest: /etc/httpd/conf.d/{{domain}}.conf
    
    - name: 'Creating Document Root'
      file:
        path: /var/www/html/{{domain}}
        state: directory
        owner: apache
        group: apache
        mode: 0755
            
    - name: 'Creating index.html'
      copy:
        content: '<h1><center>It Works</center></h1>'
        dest: /var/www/html/{{domain}}/index.html
            
    - name: 'Restarting Apache Webserver'
      service:
        name: httpd
        state: restarted
        enabled: true
            
    - name: 'Mariad-server Installing'
      yum: name=MySQL-python,mariadb-server state=present
        
    - name: 'Mariadb-server Restarting/Enabling '
      service: name=mariadb state=restarted enabled=yes
     
    - name: 'Mariadb-server Resetting Root Password'
      ignore_errors: true
      mysql_user:
        login_user: root
        login_password: ''
        name: root
        password: "{{mysql_root_password}}"
        host_all: true
            
    - name: 'Mariadb-server Removing Anonymous users'
      mysql_user:
        login_user: root
        login_password: "{{mysql_root_password}}"
        name: ''
        state: absent
        host_all: true
            
            
    - name: 'Mariadb-server Creating Additional database'
      mysql_db:
        login_user: root
        login_password: "{{mysql_root_password}}"
        name: "{{mysql_database}}"
        state: present
            
    - name: 'Mariadb-server Creating additional user/password'
      mysql_user:
        login_user: root
        login_password: "{{mysql_root_password}}"
        name: "{{mysql_user}}"
        password: "{{mysql_password}}"
        state: present
        host: localhost
        priv: "{{mysql_database}}.*:ALL"
            
    - name: 'wordpress - Downloading'
      get_url:
        url: https://wordpress.org/wordpress-4.5.8.tar.gz
        dest: /tmp/wordpress.tar.gz
            
    - name: 'Wordpress - Extraction'
      unarchive:
        src: /tmp/wordpress.tar.gz
        dest: /tmp/
        remote_src: yes
            
    - name: 'Wordpress - Copying Wordpress To DocumentRoot'
      shell: 'cp -r /tmp/wordpress/*  /var/www/html/{{domain}}'
        
    - name: 'Wordpress - Creating wp-config.php'
      template:
        src: wp-config.php.j2
        dest: /var/www/html/{{domain}}/wp-config.php
        
```

#  Creating the template files 

---
# vim virtualhost.j2
<virtualhost *:80>
    servername {{domain}}
    documentroot /var/www/html/{{domain}}
    directoryindex index.html index.php
</virtualhost>

---

# vim wp-config.php.j2

<?php

/** The name of the database for WordPress */
define( 'DB_NAME', "{{mysql_database}}" );

/** MySQL database username */
define( 'DB_USER', "{{mysql_user}}" );

/** MySQL database password */
define( 'DB_PASSWORD', "{{mysql_password}}" );

/** MySQL hostname */
define( 'DB_HOST', 'localhost' );

/** Database Charset to use in creating database tables. */
define( 'DB_CHARSET', 'utf8' );

/** The Database Collate type. Don't change this if in doubt. */
define( 'DB_COLLATE', '' );


define( 'AUTH_KEY',         'put your unique phrase here' );
define( 'SECURE_AUTH_KEY',  'put your unique phrase here' );
define( 'LOGGED_IN_KEY',    'put your unique phrase here' );
define( 'NONCE_KEY',        'put your unique phrase here' );
define( 'AUTH_SALT',        'put your unique phrase here' );
define( 'SECURE_AUTH_SALT', 'put your unique phrase here' );
define( 'LOGGED_IN_SALT',   'put your unique phrase here' );
define( 'NONCE_SALT',       'put your unique phrase here' );


$table_prefix = 'wp_';


define( 'WP_DEBUG', false );


if ( ! defined( 'ABSPATH' ) ) {
	define( 'ABSPATH', dirname( __FILE__ ) . '/' );
}

require_once( ABSPATH . 'wp-settings.php' );
    
---


