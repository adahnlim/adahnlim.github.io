---
layout: post
title:  Ubuntu에서 LDAP 서버 구성하기
description: Ubnutu LDAP 서버구성
tags: ldap

---


LDAP(Lightweight Directory Access Protocol)은 중앙에서 인증체계를 관리할 수 있는 디렉토리 프로토콜입니다.  

1. LDAP 서버 설치


		# vi /etc/hosts
		# dc=example,dc=com 
		127.0.1.1       ldap01.example.com	ldap01

		# sudo apt install slapd ldap-utils
		중간에 administartor passsword 물어보면 입력한다.
		# vi /etc/ldap/ldap.conf 
		..
		BASE    dc=example,dc=com
		URI     ldap://hostname.example.com or ldap://ldapip
		..
		# dpkg-reconfigure slapd

2. LDAP 서버 구성

	- LDAP구성 : NO 선택  
	![](https://github.com/adahnlim/adahnlim.github.io/blob/master/images/ldap-3.PNG?raw=true)

	- baseDN : /etc/hosts에서 입력한 domain
	![](https://github.com/adahnlim/adahnlim.github.io/blob/master/images/ldap-4.PNG?raw=true)

	- Organization 입력  
	![](https://github.com/adahnlim/adahnlim.github.io/blob/master/images/ldap-5.PNG?raw=true)

	- Administartor Password 입력  
	![](https://github.com/adahnlim/adahnlim.github.io/blob/master/images/ldap-6.PNG?raw=true)

	- DB선택 (HDB권장)  
	![](https://github.com/adahnlim/adahnlim.github.io/blob/master/images/ldap-7.PNG?raw=true)

	- slapd가 Purge될때 DB삭제할것인지   
	![](https://github.com/adahnlim/adahnlim.github.io/blob/master/images/ldap-8.PNG?raw=true)

	- 기존설정 이동  
	![](https://github.com/adahnlim/adahnlim.github.io/blob/master/images/ldap-9.PNG?raw=true)

	- LDAPv2 이용할것인지 (NO)  
	![](https://github.com/adahnlim/adahnlim.github.io/blob/master/images/ldap-10.PNG?raw=true)
	

			 * Stopping OpenLDAP slapd                                                                                                                            [ OK ] 
			  Moving old database directory to /var/backups:
			  - directory unknown... done.
			  Creating initial configuration... done.
			  Creating LDAP directory... done.
			 * Starting OpenLDAP slapd                                                                                                                            [ OK ] 
			Processing triggers for libc-bin (2.19-0ubuntu6.7) ...
			root@ldap01:~# 


3. LDAP 확인


		root@ldap01:~# ldapsearch -x
		# extended LDIF
		#
		# LDAPv3
		# base <dc=example,dc=com> (default) with scope subtree
		# filter: (objectclass=*)
		# requesting: ALL
		#
		
		# example.com
		dn: dc=example,dc=com
		objectClass: top
		objectClass: dcObject
		objectClass: organization
		o: example
		dc: example
		
		# admin, example.com
		dn: cn=admin,dc=example,dc=com
		objectClass: simpleSecurityObject
		objectClass: organizationalRole
		cn: admin
		description: LDAP administrator
		
		# search result
		search: 2
		result: 0 Success
		
		# numResponses: 3
		# numEntries: 2
		root@ldap01:~# 


4. Group 및 User 추가

 - a node called People (to store users)
 - a node called Groups (to store groups)
 - a group called miners
 - a user called john

			# vi add.ldif
			# People 그룹 생성
			dn: ou=People,dc=example,dc=com
			objectClass: organizationalUnit
			ou: People
			
			# Groups 그룹 생성
			dn: ou=Groups,dc=example,dc=com
			objectClass: organizationalUnit
			ou: Groups

			
			# miners in Groups 생성
			dn: cn=miners,ou=Groups,dc=example,dc=com
			objectClass: posixGroup
			cn: miners
			gidNumber: 5000
			
			# john User 생성
			dn: uid=john,ou=People,dc=example,dc=com
			objectClass: inetOrgPerson
			objectClass: posixAccount
			objectClass: shadowAccount
			uid: john
			sn: Doe
			givenName: John
			cn: John Doe
			displayName: John Doe
			uidNumber: 10000
			gidNumber: 5000
			userPassword: johnldap
			gecos: John Doe
			loginShell: /bin/bash
			homeDirectory: /home/john


			# ldapadd -x -D cn=admin,dc=example,dc=com -W -f add.ldif 
			Enter LDAP Password: 
			adding new entry "ou=People,dc=example,dc=com"
			
			adding new entry "ou=Groups,dc=example,dc=com"
			
			adding new entry "cn=miners,ou=Groups,dc=example,dc=com"
			
			adding new entry "uid=john,ou=People,dc=example,dc=com"


5. 확인  

		# ldapsearch -x
		# extended LDIF
		#
		# LDAPv3
		# base <dc=example,dc=com> (default) with scope subtree
		# filter: (objectclass=*)
		# requesting: ALL
		#
		
		# example.com
		dn: dc=example,dc=com
		objectClass: top
		objectClass: dcObject
		objectClass: organization
		o: example
		dc: example
		
		# admin, example.com
		dn: cn=admin,dc=example,dc=com
		objectClass: simpleSecurityObject
		objectClass: organizationalRole
		cn: admin
		description: LDAP administrator
		
		# People, example.com
		dn: ou=People,dc=example,dc=com
		objectClass: organizationalUnit
		ou: People
		
		# Groups, example.com
		dn: ou=Groups,dc=example,dc=com
		objectClass: organizationalUnit
		ou: Groups
		
		# miners, Groups, example.com
		dn: cn=miners,ou=Groups,dc=example,dc=com
		objectClass: posixGroup
		cn: miners
		gidNumber: 5000
		
		# john, People, example.com
		dn: uid=john,ou=People,dc=example,dc=com
		objectClass: inetOrgPerson
		objectClass: posixAccount
		objectClass: shadowAccount
		uid: john
		sn: Doe
		givenName: John
		cn: John Doe
		displayName: John Doe
		uidNumber: 10000
		gidNumber: 5000
		gecos: John Doe
		loginShell: /bin/bash
		homeDirectory: /home/joh
		
		# search result
		search: 2
		result: 0 Success
		
		# numResponses: 7
		# numEntries: 6
