---	
layout: post
title:  MySQL Dump Shell Script
description: MySQL Dump Shell Script
tags: mysql-dump mysql
---	

- Script

		#!/bin/bash
		DATE=`date +%Y%m%d`
		DATE8=`date +%Y%m%d -d -8days`

		USERNAME="root"
		PASSWORD="1"

		DATABASE="test"

		BACKUPSRC="/home/aaa/backup"
		LOGFILE="log/$DATE.log"

		echo "1. b backup" >> $LOGFILE
		mysqldump -u$USERNAME -p$PASSWORD  $DATABASE > $BACKUPSRC/$DATABASE_bak_${DATE}.sql

		echo "# File List" >> $LOGFILE
		ls $BACKUPSRC |grep $DATE >> $LOGFILE

		echo "2. backup file delete (D-8)" >>$LOGFILE
		echo "# File List" >> $LOGFILE
		ls $BACKUPSRC |grep $DATE8 >> $LOGFILE
		rm $BACKUPSRC/*$DATE8.sql -f