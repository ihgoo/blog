title: "ejabberd安装配置"
date: 2015-04-28 11:09:54
categories:  XMPP
---

##简介

官方网站：http://www.ejabberd.im/
ejabberd 是 Jabber/XMPP 协议的即时通讯服务器，使用 GPLv2许可（自由和开放源码） ，基于 erlang/otp 开发。其它特性还包括， 跨平台，容错， clusterable和模块化。

<!--more-->
##下载&安装
这里是debain6 64位的系统，所以下载对应的

	wget https://www.process-one.net/downloads/ejabberd/15.04/ejabberd-15.04-linux-x86_64-installer.run

arm架构的可以下载这个

	wget https://www.process-one.net/downloads/downloads-action.php?file=/ejabberd/15.04/ejabberd-15.04-linux-armhf-installer.run

下载完成后需提权，否则无法运行安装包

	chmod 777 ejabberd-15.04-linux-x86_64-installer.run

安装

	./ejabberd-15.04-linux-x86_64-installer.run


一路next，设置服务器域，设置管理员帐号及密码，集群配置，节点名称等。


将/opt/ejabberd-15.03/bin配置到环境变量中(/etc/profile)

支持以下命令：

	Usage: ejabberdctl [--node nodename] [--auth user host password] command [options]

		   backup file
		         Store the database to backup file

		   connected_users
		         List all established sessions

		   connected_users_number
		         Get the number of established sessions

		   convert_to_yaml in out
		         Convert the input file from Erlang to YAML format

		   delete_expired_messages
		         Delete expired offline messages from database

		   delete_old_messages days
		         Delete offline messages older than DAYS

		   dump file
		         Dump the database to text file

		   dump_table file table
		         Dump a table to text file

		   export2odbc host directory
		         Export virtual host information from Mnesia tables to SQL files

		   export_odbc host file
		         Export all tables as SQL queries to a file

		   export_piefxis dir
		         Export data of all users in the server to PIEFXIS files (XEP-0227)

		   export_piefxis_host dir host
		         Export data of users in a host to PIEFXIS files (XEP-0227)

		   get_loglevel
		         Get the current loglevel

		   help [--tags [tag] | com?*]
		         Show help (try: ejabberdctl help help)

		   import_dir file
		         Import users data from jabberd14 spool dir

		   import_file file
		         Import user data from jabberd14 spool file

		   import_piefxis file
		         Import users data from a PIEFXIS file (XEP-0227)

		   incoming_s2s_number
		         Number of incoming s2s connections on the node

		   install_fallback file
		         Install the database from a fallback file

		   kick_user user host
		         Disconnect user's active sessions

		   load file
		         Restore the database from text file

		   mnesia [info]
		         show information of Mnesia system

		   mnesia_change_nodename oldnodename newnodename oldbackup newbackup
		         Change the erlang node name in a backup file

		   module_check module


		   module_install module


		   module_uninstall module


		   module_upgrade module


		   modules_available


		   modules_installed


		   modules_update_specs


		   outgoing_s2s_number
		         Number of outgoing s2s connections on the node

		   register user host password
		         Register a user

		   registered_users host
		         List all registered users in HOST

		   registered_vhosts
		         List all registered vhosts in SERVER

		   reload_config
		         Reload ejabberd configuration file into memory

		   rename_default_nodeplugin
		         Update PubSub table from old ejabberd trunk SVN to 2.1.0

		   reopen_log
		         Reopen the log files

		   restart
		         Restart ejabberd

		   restore file
		         Restore the database from backup file

		   set_master nodename
		         Set master node of the clustered Mnesia tables

		   status
		         Get ejabberd status

		   stop
		         Stop ejabberd

		   stop_kindly delay announcement
		         Inform users and rooms, wait, and stop the server

		   unregister user host
		         Unregister a user

		   update module
		         Update the given module, or use the keyword: all

		   update_list
		         List modified modules that can be updated

		   user_resources user host
		         List user's connected resources


浏览器输入 域名:5280/admin 进入ejabberd管理页面。简单的安装已经完成了。