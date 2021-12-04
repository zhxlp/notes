# FreeBSD7.0安装AMP

### 环境

* 操作系统 FreeBSD 7.0

### 要求

* Apache 2.4.48
* Mysql 4.0.27
* php 5.6.40
* samba 3.6.25
* git 2.32.0

### Mysql安装及配置

*   查找 gcc-3.4 安装包名称

    ```
    cd /usr/ports/make search name=gcc-3.4​# Port: gcc-3.4.6_2,1# Path: /usr/ports/lang/gcc34# Info: GNU Compiler Collection 3.4# Maint:    gerald@FreeBSD.org# B-deps:   bison-2.3_3,1 gettext-0.16.1_3 gmake-3.81_2 libiconv-1.11_1 m4-1.4.9,1 perl-5.8.8_1# R-deps:   libiconv-1.11_1# WWW:  http://gcc.gnu.org/
    ```
*   安装 `gcc-3.4.6_2,1`

    ```
    # 配置 package 源setenv PACKAGESITE ftp://ftp-archive.freebsd.org/pub/FreeBSD-Archive/old-releases/i386/7.0-RELEASE/packages/All/# 安装 gcc-3.4pkg_add -r gcc-3.4.6_2,1
    ```
*   查找 `mysql-server-4.0`

    ```
    cd /usr/ports/make search name=mysql-server-4.0​# Port: mysql-server-4.0.27# Path: /usr/ports/databases/mysql40-server# Info: Multithreaded SQL database (server)# Maint:    ale@FreeBSD.org# B-deps:   libtool-1.5.24 mysql-client-4.0.27# R-deps:   mysql-client-4.0.27# WWW:  http://www.mysql.com/
    ```
*   下载 `mysql-4.0.27.tar.gz` 到 `/usr/ports/distfiles/`

    ```
    wget -O /usr/ports/distfiles/mysql-4.0.27.tar.gz http://ftp.linux.co.kr/pub/mysql/mysql-4.0.27.tar.gz
    ```
*   修改 `/usr/ports/databases/mysql40-server/Makefile`文件，注释如下内容

    ```
    # .if ${OSVERSION} >= 700000# IGNORE=         obsolete and does not build with gcc4.2; use mysql 5 or later# .endif
    ```
*   编译安装mysql

    ```
    cd /usr/ports/databases/mysql40-servermake CC=gcc34 CXX=g++34 install
    ```
*   开启 mysql 服务

    编辑 `/etc/rc.conf`文件，添加 `mysql_enable="YES"`
*   启动mysql

    ```
    /usr/local/etc/rc.d/mysql-server start
    ```
*   设置 `root` 密码

    ```
    mysqladmin -u root password '123456'
    ```
*   添加 `root@%` 用户

    ```
    GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY '123456' WITH GRANT OPTION ;
    ```

### Apache安装及配置

*   expat

    源码下载地址：`https://github.com/libexpat/libexpat/releases/download/R_2_2_10/expat-2.2.10.tar.gz`

    ```
    tar -xvf expat-2.2.10.tar.gzcd expat-2.2.10./configure --prefix=/usr/local/expat-2.2.10make && make installcd ..
    ```
*   apr

    源码下载地址：`https://apache.claz.org//apr/apr-1.7.0.tar.gz`

    ```
    tar -xvf apr-1.7.0.tar.gzcd apr-1.7.0./configure --prefix=/usr/local/apr-1.7.0make && make installcd ..
    ```
*   apr-util

    源码下载地址：`https://apache.claz.org//apr/apr-util-1.6.1.tar.gz`

    ```
    tar -xvf apr-util-1.6.1.tar.gzcd apr-util-1.6.1./configure --prefix=/usr/local/apr-util-1.6.1 --with-apr=/usr/local/apr-1.7.0 --with-expat=/usr/local/expat-2.2.10make && make installcd ..
    ```
*   pcre

    源码下载地址：`https://ftp.pcre.org/pub/pcre/pcre-8.44.tar.gz`

    ```
    tar -xvf pcre-8.44.tar.gzcd pcre-8.44./configure --prefix=/usr/local/pcre-8.44make && make installcd ..
    ```
*   perl

    源码下载地址：`https://www.cpan.org/src/5.0/perl-5.18.4.tar.gz`

    ```
    tar -xvf perl-5.18.4.tar.gzcd perl-5.18.4./Configure -des -Dprefix=/usr/local/perl-5.18.4make && make installcd ..
    ```
*   openssl

    源码下载地址：`https://www.openssl.org/source/openssl-1.1.1k.tar.gz`

    ```
    tar -xvf openssl-1.1.1k.tar.gzcd openssl-1.1.1k./config --prefix=/usr/local/openssl-1.1.1kmake && make installcd ..
    ```
*   配置 openssl 动态库

    ```
    echo '/usr/local/openssl-1.1.1k/lib' > /usr/local/libdata/ldconfig/openssl-1.1.1k/etc/rc.d/ldconfig restart
    ```
*   httpd

    源码下载地址：`https://mirrors.ocf.berkeley.edu/apache//httpd/httpd-2.4.48.tar.gz`

    ```
    tar -xvf httpd-2.4.48.tar.gzcd httpd-2.4.48./configure --prefix=/usr/local/apache24 --with-apr=/usr/local/apr-1.7.0 \--with-apr-util=/usr/local/apr-util-1.6.1 --with-pcre=/usr/local/pcre-8.44 \--enable-ssl --with-ssl=/usr/local/openssl-1.1.1k \--enable-so  --enable-cgi --enable-rewrite --with-zlib \--enable-modules=mostmake && make installcd ..
    ```
*   配置 rc.d

    编辑 `/usr/local/etc/rc.d/apache24`

    ```
    #!/bin/sh## PROVIDE: apache24# REQUIRE: LOGIN cleanvar sshd# KEYWORD: shutdown​## Add the following lines to /etc/rc.conf to enable apache24:# apache24_enable (bool):      Set to "NO" by default.#                             Set it to "YES" to enable apache24# apache24_profiles (str):     Set to "" by default.#                              Define your profiles here.# apache24limits_enable (bool):Set to "NO" by default.#                             Set it to yes to run `limits $limits_args`#                             just before apache starts.# apache24_flags (str):        Set to "" by default.#                             Extra flags passed to start command.# apache24limits_args (str):   Default to "-e -C daemon"#                             Arguments of pre-start limits run.# apache24_http_accept_enable (bool): Set to "NO" by default.#                             Set to yes to check for accf_http kernel#                             module on start up and load if not loaded.# apache24_fib (str):         Set an altered default network view for apache​. /etc/rc.subr​name="apache24"rcvar=apache24_enable​apache24_prefix="/usr/local/apache24"​start_precmd="apache24_prestart"restart_precmd="apache24_checkconfig"reload_precmd="apache24_checkconfig"reload_cmd="apache24_graceful"graceful_cmd="apache24_graceful"gracefulstop_cmd="apache24_gracefulstop"configtest_cmd="apache24_checkconfig"command="${apache24_prefix}/bin/httpd"_pidprefix="${apache24_prefix}/logs/httpd"pidfile="${_pidprefix}.pid"required_files="${apache24_prefix}/conf/httpd.conf"envvars="${apache24_prefix}/bin/envvars"​[ -z "$apache24_enable" ]       && apache24_enable="NO"[ -z "$apache24limits_enable" ] && apache24limits_enable="NO"[ -z "$apache24limits_args" ]   && apache24limits_args="-e -C daemon"[ -z "$apache24_http_accept_enable" ] && apache24_http_accept_enable="NO"​apache24_accf(){  if checkyesno apache24_http_accept_enable; then    /sbin/kldstat -v | grep accf_http > /dev/null 2>&1 || /sbin/kldload accf_http || return ${?}    /sbin/kldstat -v | grep accf_data > /dev/null 2>&1 || /sbin/kldload accf_data || return ${?}  else    apache24_flags="${apache24_flags} -DNOHTTPACCEPT"  fi}​load_rc_config $name​if [ -n "$2" ]; then    profile="$2"    if [ "x${apache24_profiles}" != "x" ]; then        pidfile="${_pidprefix}.${profile}.pid"        eval apache24_configfile="\${apache24_${profile}_configfile:-}"        if [ "x${apache24_configfile}" = "x" ]; then            echo "You must define a configuration file (apache24_${profile}_configfile)"            exit 1        fi        required_files="${apache24_configfile}"        eval apache24_enable="\${apache24_${profile}_enable:-${apache24_enable}}"        eval apache24_flags="\${apache24_${profile}_flags:-${apache24_flags}}"        eval apache24_http_accept_enable="\${apache24_${profile}_http_accept_enable:-${apache24_http_accept_enable}}"        eval apache24limits_enable="\${apache24limits_${profile}_enable:-${apache24limits_enable}}"        eval apache24limits_args="\${apache24limits_${profile}_args:-${apache24limits_args}}"        eval apache24_fib="\${apache24_${profile}_fib:-${apache24_fib}}"        eval command="\${apache24_${profile}_command:-${command}}"        eval pidfile="\${apache24_${profile}_pidfile:-${pidfile}}"        eval apache24_envvars="\${apache24_${profile}_envvars:-${envvars}}"        apache24_flags="-f ${apache24_configfile} -c \"PidFile ${pidfile}\" ${apache24_flags}"    else        echo "$0: extra argument ignored"    fielse    eval apache24_envvars=${envvars}    if [ "x${apache24_profiles}" != "x" -a "x$1" != "x" ]; then        for profile in ${apache24_profiles}; do            eval _enable="\${apache24_${profile}_enable}"            case "x${_enable:-${apache24_enable}}" in            x|x[Nn][Oo]|x[Nn][Oo][Nn][Ee])                continue                ;;            x[Yy][Ee][Ss])                ;;            *)                if test -z "$_enable"; then                    _var=apache24_enable                else                    _var=apache24_"${profile}"_enable                fi                echo "Bad value" \                    "'${_enable:-${apache24_enable}}'" \                    "for ${_var}. " \                    "Profile ${profile} skipped."                continue                ;;            esac            echo "===> apache24 profile: ${profile}"            /usr/local/etc/rc.d/apache24 $1 ${profile}            retcode="$?"            if [ "0${retcode}" -ne 0 ]; then                failed="${profile} (${retcode}) ${failed:-}"            else                success="${profile} ${success:-}"            fi        done        exit 0    fifi​if [ "${1}" != "stop" ] ; then \    apache24_accffi​apache24_requirepidfile(){    apache24_checkconfig​    if [ ! "0`check_pidfile ${pidfile} ${command}`" -gt 1 ]; then        echo "${name} not running? (check $pidfile)."        exit 1    fi}​apache24_checkconfig(){    if test -f ${apache24_envvars}    then        . ${apache24_envvars}    fi​    echo "Performing sanity check on apache24 configuration:"    eval ${command} ${apache24_flags} -t}​apache24_graceful() {    apache24_requirepidfile​    echo "Performing a graceful restart"    eval ${command} ${apache24_flags} -k graceful}​apache24_gracefulstop() {    apache24_requirepidfile​    echo "Performing a graceful stop"    eval ${command} ${apache24_flags} -k graceful-stop}​apache24_precmd(){    apache24_checkconfig​    if checkyesno apache24limits_enable    then        eval `/usr/bin/limits ${apache24limits_args}` 2>/dev/null    else        return 0        fi​}​apache24_checkfib () {    if command -v check_namevarlist > /dev/null 2>&1; then        check_namevarlist fib && return 0    fi​    $SYSCTL net.fibs >/dev/null 2>&1 || return 0​    apache24_fib=${apache24_fib:-"NONE"}    if [ "x$apache24_fib" != "xNONE" ]    then        command="/usr/sbin/setfib -F ${apache24_fib} ${command}"    else        return 0    fi}​apache24_prestart() {    apache24_checkfib    apache24_precmd}​extra_commands="reload graceful gracefulstop configtest"run_rc_command "$1"​
    ```

    添加执行权限

    ```
    chmod +x /usr/local/etc/rc.d/apache24
    ```
*   配置 `httpd.conf`

    编辑`/usr/local/apache24/conf/httpd.conf`，修改 `ServerName 127.0.0.1`
*   启动 apache

    编辑 `/etc/rc.conf` 添加 `apache24_enable="YES"`和`apache24_http_accept_enable="YES"`

    启动 apache

    ```
    /usr/local/etc/rc.d/apache24 start
    ```

### php安装及配置

*   openssl

    源码下载地址：`https://www.openssl.org/source/openssl-0.9.8g.tar.gz`

    ```
    tar -xvf openssl-0.9.8g.tar.gzcd openssl-0.9.8g./config --prefix=/usr/local/openssl-0.9.8gmake && make installcd ..
    ```
*   libxml2

    源码下载地址：`ftp://xmlsoft.org/libxml2/libxml2-2.7.8.tar.gz`

    ```
    tar -xvf libxml2-2.7.8.tar.gzcd libxml2-2.7.8./configure --prefix=/usr/local/libxml2-2.7.8make && make installcd ..
    ```
*   php

    源码下载地址：`https://www.php.net/distributions/php-5.6.40.tar.gz`

    ```
    tar xvf php-5.6.40.tar.gzcd php-5.6.40./configure --prefix=/usr/local/php-5.6.40 --with-openssl=/usr/local/openssl-0.9.8g \--with-libxml-dir=/usr/local/libxml2-2.7.8 \--without-pdo-sqlite --without-sqlite3 --with-mysql=/usr/local \--with-apxs2=/usr/local/apache24/bin/apxsmake && make installcp php.ini-production /usr/local/php-5.6.40/lib/php.inicd ..
    ```
*   配置 `httpd.conf`

    编辑`/usr/local/apache24/conf/httpd.conf`，修改`DirectoryIndex index.php index.html` 并添加 如下内容

    ```
    <IfModule php5_module>AddType application/x-httpd-php .php</IfModule>
    ```

### xdebug安装及配置

*   m4

    源码下载地址：`https://ftp.gnu.org/gnu/m4/m4-1.4.18.tar.gz`

    ```
    tar -xvf m4-1.4.18.tar.gzcd m4-1.4.18./configure --prefix=/usr/local/m4-1.4.18make && make installcd ..
    ```
*   autoconf

    源码下载地址：`https://ftp.gnu.org/gnu/autoconf/autoconf-2.69.tar.gz`

    ```
    tar -xvf autoconf-2.69.tar.gzcd autoconf-2.69setenv M4 /usr/local/m4-1.4.18/bin/m4./configure --prefix=/usr/local/autoconf-2.69make && make installcd ..
    ```
*   xdebug

    源码下载地址：`https://xdebug.org/files/xdebug-2.5.4.tgz`

    ```
    tar xvf xdebug-2.5.4.tgzcd xdebug-2.5.4setenv PHP_AUTOCONF /usr/local/autoconf-2.69/bin/autoconfsetenv PHP_AUTOHEADER /usr/local/autoconf-2.69/bin/autoheader/usr/local/php-5.6.40/bin/phpize./configure --enable-xdebug --with-php-config=/usr/local/php-5.6.40/bin/php-configmake && make installcd ..
    ```
*   配置 `php.ini`启用 `xdebug`

    编辑 `/usr/local/php-5.6.40/lib/php.ini`,添加如下

    ```
    [xdebug]zend_extension=xdebug.soxdebug.remote_enable = 1xdebug.remote_host = 192.168.187.1xdebug.remote_port = 9000xdebug.idekey=PHPSTORM
    ```

### php-gd安装及配置

*   libpng

    源码下载地址 `https://download.sourceforge.net/libpng/libpng-1.6.37.tar.gz`

    ```
    tar xvf libpng-1.6.37.tar.gzcd libpng-1.6.37./configure --prefix=/usr/local/libpng-1.6.37 --enable-shared  --enable-staticmake && make installcd ..
    ```
*   libjpeg

    源码下载地址 `https://download.sourceforge.net/libjpeg/6b/jpegsrc.v6b.tar.gz`

    ```
    tar xvf jpegsrc.v6b.tar.gzcd jpeg-6b./configure --prefix=/usr/local/jpeg-6b --enable-shared --enable-staticmkdir -p /usr/local/jpeg-6b/bin/mkdir -p /usr/local/jpeg-6b/man/man1/mkdir -p /usr/local/jpeg-6b/include/mkdir -p /usr/local/jpeg-6b/lib/make && make installcd ..
    ```
*   zlib

    源码下载地址 `http://www.zlib.net/zlib-1.2.11.tar.gz`

    ```
    tar -xvf zlib-1.2.11.tar.gzcd zlib-1.2.11./configure --prefix=/usr/local/zlib-1.2.11make && make installcd ..
    ```
*   FreeType

    源码下载地址`https://download.savannah.gnu.org/releases/freetype/freetype-2.10.0.tar.gz`

    ```
    tar -xvf freetype-2.10.0.tar.gzcd freetype-2.10.0./configure --prefix=/usr/local/freetype-2.10.0 --enable-freetype-config --enable-shared  --enable-staticgmake && gmake installcd ..
    ```
*   gd

    ```
    cd php-5.6.40/ext/gdsetenv PHP_AUTOCONF /usr/local/autoconf-2.69/bin/autoconfsetenv PHP_AUTOHEADER /usr/local/autoconf-2.69/bin/autoheadersetenv CPPFLAGS -I/usr/local/includesetenv LDFLAGS -L/usr/local/lib/usr/local/php-5.6.40/bin/phpize./configure --with-php-config=/usr/local/php-5.6.40/bin/php-config --with-png-dir=/usr/local/libpng-1.6.37 --with-jpeg-dir=/usr/local/jpeg-6b --with-zlib-dir=/usr/local/zlib-1.2.11 --with-freetype-dir=/usr/local/freetype-2.10.0make && make installcd ..
    ```
* 配置 `php.ini` ,添加 `extension=gd.so`

### samba安装及配置

*   samba

    源码下载地址：`https://download.samba.org/pub/samba/stable/samba-3.6.25.tar.gz`

    ```
    tar xvf samba-3.6.25.tar.gzcd samba-3.6.25/source3/./configure --prefix=/usr/local/samba-3.6.25make && make installcd ..
    ```
*   配置 动态库

    ```
    echo '/usr/local/samba-3.6.25/lib' > /usr/local/libdata/ldconfig/samba/etc/rc.d/ldconfig restart
    ```
*   配置 smb.conf

    配置 samba 匿名访问，编辑 `/usr/local/samba-3.6.25/lib/smb.conf`文件

    ```
    [global]    null passwords = yes    guest account = root    security = share		[www]    comment = www    path = /usr/local/www    public = yes    guest ok = yes    writable = yes
    ```
*   配置 rc.d

    编辑 `/usr/local/etc/rc.d/samba`

    ```
    #!/bin/sh# PROVIDE: nmbd smbd# PROVIDE: winbindd# REQUIRE: NETWORKING SERVERS DAEMON ldconfig resolv# REQUIRE: cupsd# BEFORE: LOGIN# KEYWORD: shutdown## Add the following lines to /etc/rc.conf.local or /etc/rc.conf# to enable this service:##samba_enable="YES"# or, for fine grain control:#nmbd_enable="YES"#smbd_enable="YES"# You need to enable winbindd separately, by adding:#winbindd_enable="YES"## Configuration file can be set with:#samba_config="/usr/local/samba-3.6.25/lib/smb.conf"#. /etc/rc.subrname="samba"rcvar=`set_rcvar`samba_prefix="/usr/local/samba-3.6.25"load_rc_config "${name}"# Custom commandsextra_commands="reload status"start_precmd="samba_start_precmd"start_cmd="samba_cmd"stop_cmd="samba_cmd"status_cmd="samba_cmd"restart_precmd="samba_checkconfig"reload_precmd="samba_checkconfig"reload_cmd="samba_reload_cmd"rcvar_cmd="samba_rcvar_cmd"# Defaultssamba_enable="${samba_enable:=NO}"samba_config_default="${samba_prefix}/lib/smb.conf"samba_config="${samba_config=${samba_config_default}}"command_args="${samba_config:+-s "${samba_config}"}"			#"samba_daemons="nmbd smbd"samba_daemons="${samba_daemons} winbindd"testparm_command="${samba_prefix}/bin/testparm"smbcontrol_command="${samba_prefix}/bin/smbcontrol"# Fetch parameters from configuration filesamba_parm="${testparm_command} -s -v --parameter-name"samba_idmap=$(${samba_parm} 'idmap uid' ${samba_config:+"${samba_config}"} 2>/dev/null)samba_lockdir=$(${samba_parm} 'lock directory' ${samba_config:+"${samba_config}"} 2>/dev/null)# Setup dependent variablesif [ -n "${rcvar}" ] && checkyesno "${rcvar}"; then    nmbd_enable="${nmbd_enable=YES}"    smbd_enable="${smbd_enable=YES}"    # Check that winbindd is actually configured    if [ -n "${samba_idmap}" ]; then	winbindd_enable="${winbindd_enable=YES}"    fifi# Hack to work around name change of pid file with non-default configpid_extra=if [ -n "${samba_config}" -a "${samba_config}" != "${samba_config_default}" ]; then    pid_extra="-$(basename "${samba_config}")"fi# Hack to enable check of dependent variableseval real_${rcvar}="\${${rcvar}:=NO}"	${rcvar}="YES"# Defaults for dependent variablesnmbd_enable="${nmbd_enable:=NO}"nmbd_flags="${nmbd_flags=\"-D\"}"smbd_enable="${smbd_enable:=NO}"smbd_flags="${smbd_flags=\"-D\"}"winbindd_enable="${winbindd_enable:=NO}"winbindd_flags="${winbindd_flags=''}"if [ -n "${samba_lockdir}" -a ! -e "${samba_lockdir}" ]; then    mkdir -p "${samba_lockdir}"fi# Requirementsrequired_files="${samba_config}"required_dirs="${samba_lockdir}"samba_checkconfig() {    echo -n "Performing sanity check on Samba configuration: "    if "${testparm_command}" -s ${samba_config:+"${samba_config}"} >/dev/null 2>&1; then	echo "OK"    else	echo "FAILED"	return 1    fi}samba_start_precmd() {    # XXX: Never delete winbindd_idmap, winbindd_cache and group_mapping    if [ -n "${samba_lockdir}" -a -d "${samba_lockdir}" ]; then	echo -n "Removing stale Samba tdb files: "	for file in brlock.tdb browse.dat connections.tdb gencache.tdb \		    locking.tdb messages.tdb namelist.debug sessionid.tdb \		    unexpected.tdb	do	    rm "${samba_lockdir}/${file}" </dev/null 2>/dev/null && echo -n '.'	done	echo " done"    fi}samba_rcvar_cmd() {    # Prevent recursive calling    unset "${rc_arg}_cmd" "${rc_arg}_precmd" "${rc_arg}_postcmd"    # Check master variable    echo "# ${name}"    if [ -n "${rcvar}" ]; then	# Use original configured value	if checkyesno "real_${rcvar}"; then	    echo "\$${rcvar}=YES"	else	    echo "\$${rcvar}=NO"	fi    fi    # Check dependent variables    samba_cmd "${_rc_prefix}${rc_arg}" ${rc_extra_args}}samba_reload_cmd() {    local name rcvar command pidfile    # Prevent recursive calling    unset "${rc_arg}_cmd" "${rc_arg}_precmd" "${rc_arg}_postcmd"    # Apply to all the daemons    for name in ${samba_daemons}; do    	rcvar=$(set_rcvar)	command="${samba_prefix}/sbin/${name}"	pidfile="${samba_lockdir}/${name}${pid_extra}.pid"	# Daemon should be enabled and running	if [ -n "${rcvar}" ] && checkyesno "${rcvar}"; then	    if [ -n "$(check_pidfile "${pidfile}" "${command}")" ]; then		debug "reloading ${name} configuration"		echo "Reloading ${name}."		# XXX: Hack with pid_extra		"${smbcontrol_command}" "${name}${pid_extra}" 'reload-config' ${command_args} >/dev/null 2>&1	    fi	fi    done}samba_cmd() {    local name rcvar command pidfile samba_daemons    # Prevent recursive calling    unset "${rc_arg}_cmd" "${rc_arg}_precmd" "${rc_arg}_postcmd"    # Stop processes in the reverse to order    if [ "${rc_arg}" = "stop" ] ; then	samba_daemons=$(reverse_list ${samba_daemons})    fi    # Apply to all the daemons    for name in ${samba_daemons}; do	rcvar=$(set_rcvar)	command="${samba_prefix}/sbin/${name}"	pidfile="${samba_lockdir}/${name}${pid_extra}.pid"		run_rc_command "${_rc_prefix}${rc_arg}" ${rc_extra_args}    done}run_rc_command "$1"
    ```

    添加执行权限

    ```
    chmod +x /usr/local/etc/rc.d/samba
    ```
*   启动 samba

    编辑 `/etc/rc.conf` 添加 `samba_enable="YES"`

    启动 samba

    ```
    /usr/local/etc/rc.d/samba start
    ```

### git安装及配置

*   gmake

    ```
    # 配置 package 源setenv PACKAGESITE ftp://ftp-archive.freebsd.org/pub/FreeBSD-Archive/old-releases/i386/7.0-RELEASE/packages/All/# 安装 gmakepkg_add -r gmake-3.81_2
    ```
*   git

    源代码下载地址：`https://mirrors.edge.kernel.org/pub/software/scm/git/git-2.32.0.tar.gz`

    ```
    tar -xvf git-2.32.0.tar.gzcd git-2.32.0./configure --prefix=/usr/local/git-2.32.0gmake && gmake installln -sf /usr/local/git-2.32.0/bin/git /usr/bin/gitcd ..
    ```

### 配置vhosts

*   配置 `httpd.conf`

    编辑`/usr/local/apache24/conf/httpd.conf`,取消 `Include conf/extra/httpd-vhosts.conf`注释
*   配置 `httpd-vhosts`

    编辑 `/usr/local/apache24/conf/extra/httpd-vhosts.conf`

    ```
    <VirtualHost *:80>    ServerName localhost    DocumentRoot "/usr/local/www/test/"    <Directory "/usr/local/www/test/">        Options Indexes FollowSymLinks        AllowOverride All        Order allow,deny        Allow from all        Require all granted    </Directory></VirtualHost>
    ```
*   创建php文件

    创建 php 代码目录

    ```
    mkdir -p /usr/local/www/test/
    ```

    创建 `/usr/local/www/test/index.php` 文件，内容如下

    ```
    <?phpphpinfo();?>
    ```
*   重启 apache服务

    ```
    /usr/local/etc/rc.d/apache24 restart
    ```
*   其它

    ```
    取消 `LoadModule rewrite_module modules/mod_rewrite.so`注释取消 `LoadModule ssl_module modules/mod_ssl.so`注释
    ```

\
