---
layout: page
title: "Apache Shiro Authentication for Apache Zeppelin"
description: "Apache Shiro is a powerful and easy-to-use Java security framework that performs authentication, authorization, cryptography, and session management. This document explains step by step how Shiro can be used for Zeppelin notebook authentication."
group: setup/security
---
<!--
Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
-->
{% include JB/setup %}

# Apache Shiro authentication for Apache Zeppelin

<div id="toc"></div>

## Overview
[Apache Shiro](http://shiro.apache.org/) is a powerful and easy-to-use Java security framework that performs authentication, authorization, cryptography, and session management. In this documentation, we will explain step by step how Shiro works for Zeppelin notebook authentication.

When you connect to Apache Zeppelin, you will be asked to enter your credentials. Once you logged in, then you have access to all notes including other user's notes.

## Important Note
By default, Zeppelin allows anonymous access. It is strongly recommended that you consider setting up Apache Shiro for authentication (as described in this document, see 2 Secure the Websocket channel), or only deploy and use Zeppelin in a secured and trusted environment.

## Security Setup
You can setup **Zeppelin notebook authentication** in some simple steps.

### 1. Enable Shiro
By default in `conf`, you will find `shiro.ini.template`, this file is used as an example and it is strongly recommended
to create a `shiro.ini` file by doing the following command line

```bash
cp conf/shiro.ini.template conf/shiro.ini
```

For the further information about  `shiro.ini` file format, please refer to [Shiro Configuration](http://shiro.apache.org/configuration.html#Configuration-INISections).

### 2. Start Zeppelin

```bash
bin/zeppelin-daemon.sh start #(or restart)
```

Then you can browse Zeppelin at [http://localhost:8080](http://localhost:8080).

### 3. Login
Finally, you can login using one of the below **username/password** combinations.

<center><img src="{{BASE_PATH}}/assets/themes/zeppelin/img/docs-img/zeppelin-login.png"></center>

```
[users]

admin = password1, admin
user1 = password2, role1, role2
user2 = password3, role3
user3 = password4, role2
```
You can set the roles for each users next to the password.

## Groups and permissions (optional)
In case you want to leverage user groups and permissions, use one of the following configuration for LDAP or AD under `[main]` segment in `shiro.ini`.

```
activeDirectoryRealm = org.apache.zeppelin.realm.ActiveDirectoryGroupRealm
activeDirectoryRealm.systemUsername = userNameA
activeDirectoryRealm.systemPassword = passwordA
activeDirectoryRealm.searchBase = CN=Users,DC=SOME_GROUP,DC=COMPANY,DC=COM
activeDirectoryRealm.url = ldap://ldap.test.com:389
activeDirectoryRealm.groupRolesMap = "CN=aGroupName,OU=groups,DC=SOME_GROUP,DC=COMPANY,DC=COM":"group1"
activeDirectoryRealm.authorizationCachingEnabled = false
activeDirectoryRealm.principalSuffix = @corp.company.net

ldapRealm = org.apache.zeppelin.realm.LdapGroupRealm
# search base for ldap groups (only relevant for LdapGroupRealm):
ldapRealm.contextFactory.environment[ldap.searchBase] = dc=COMPANY,dc=COM
ldapRealm.contextFactory.url = ldap://ldap.test.com:389
ldapRealm.userDnTemplate = uid={0},ou=Users,dc=COMPANY,dc=COM
ldapRealm.contextFactory.authenticationMechanism = simple
```

also define roles/groups that you want to have in system, like below;

```
[roles]
admin = *
hr = *
finance = *
group1 = *
```

## Configure Realm (optional)
Realms are responsible for authentication and authorization in Apache Zeppelin. By default, Apache Zeppelin uses [IniRealm](https://shiro.apache.org/static/latest/apidocs/org/apache/shiro/realm/text/IniRealm.html) (users and groups are configurable in `conf/shiro.ini` file under `[user]` and `[group]` section). You can also leverage Shiro Realms like [JndiLdapRealm](https://shiro.apache.org/static/latest/apidocs/org/apache/shiro/realm/ldap/JndiLdapRealm.html), [JdbcRealm](https://shiro.apache.org/static/latest/apidocs/org/apache/shiro/realm/jdbc/JdbcRealm.html) or create [our own](https://shiro.apache.org/static/latest/apidocs/org/apache/shiro/realm/AuthorizingRealm.html).
To learn more about Apache Shiro Realm, please check [this documentation](http://shiro.apache.org/realm.html).

We also provide community custom Realms.

**Note**: When using any of the below realms the default 
      password-based (IniRealm) authentication needs to be disabled.

### Active Directory

```
activeDirectoryRealm = org.apache.zeppelin.realm.ActiveDirectoryGroupRealm
activeDirectoryRealm.systemUsername = userNameA
activeDirectoryRealm.systemPassword = passwordA
activeDirectoryRealm.hadoopSecurityCredentialPath = jceks://file/user/zeppelin/conf/zeppelin.jceks
activeDirectoryRealm.searchBase = CN=Users,DC=SOME_GROUP,DC=COMPANY,DC=COM
activeDirectoryRealm.url = ldap://ldap.test.com:389
activeDirectoryRealm.groupRolesMap = "CN=aGroupName,OU=groups,DC=SOME_GROUP,DC=COMPANY,DC=COM":"group1"
activeDirectoryRealm.authorizationCachingEnabled = false
activeDirectoryRealm.principalSuffix = @corp.company.net
```


Also instead of specifying systemPassword in clear text in shiro.ini administrator can choose to specify the same in "hadoop credential".
Create a keystore file using the hadoop credential commandline, for this the hadoop commons should be in the classpath
`hadoop credential create activeDirectoryRealm.systempassword -provider jceks://file/user/zeppelin/conf/zeppelin.jceks`

Change the following values in the Shiro.ini file, and uncomment the line:
`activeDirectoryRealm.hadoopSecurityCredentialPath = jceks://file/user/zeppelin/conf/zeppelin.jceks`

### LDAP

Two options exist for configuring an LDAP Realm. The simpler to use is the LdapGroupRealm. How ever it has limited
flexibility with mapping of ldap groups to users and for authorization for user groups. A sample configuration file for
this realm is given below.

```
ldapRealm = org.apache.zeppelin.realm.LdapGroupRealm
# search base for ldap groups (only relevant for LdapGroupRealm):
ldapRealm.contextFactory.environment[ldap.searchBase] = dc=COMPANY,dc=COM
ldapRealm.contextFactory.url = ldap://ldap.test.com:389
ldapRealm.userDnTemplate = uid={0},ou=Users,dc=COMPANY,dc=COM
ldapRealm.contextFactory.authenticationMechanism = simple
```

The other more flexible option is to use the LdapRealm. It allows for mapping of ldapgroups to roles and also allows for
 role/group based authentication into the zeppelin server. Sample configuration for this realm is given below.

```
[main]
ldapRealm=org.apache.zeppelin.realm.LdapRealm

ldapRealm.contextFactory.authenticationMechanism=simple
ldapRealm.contextFactory.url=ldap://localhost:33389
ldapRealm.userDnTemplate=uid={0},ou=people,dc=hadoop,dc=apache,dc=org
# Ability to set ldap paging Size if needed default is 100
ldapRealm.pagingSize = 200
ldapRealm.authorizationEnabled=true
ldapRealm.contextFactory.systemAuthenticationMechanism=simple
ldapRealm.searchBase=dc=hadoop,dc=apache,dc=org
ldapRealm.userSearchBase = dc=hadoop,dc=apache,dc=org
ldapRealm.groupSearchBase = ou=groups,dc=hadoop,dc=apache,dc=org
ldapRealm.groupObjectClass=groupofnames
# Allow userSearchAttribute to be customized
ldapRealm.userSearchAttributeName = sAMAccountName
ldapRealm.memberAttribute=member
# force usernames returned from ldap to lowercase useful for AD
ldapRealm.userLowerCase = true
# ability set searchScopes subtree (default), one, base
ldapRealm.userSearchScope = subtree;
ldapRealm.groupSearchScope = subtree;
ldapRealm.memberAttributeValueTemplate=cn={0},ou=people,dc=hadoop,dc=apache,dc=org
ldapRealm.contextFactory.systemUsername=uid=guest,ou=people,dc=hadoop,dc=apache,dc=org
ldapRealm.contextFactory.systemPassword=S{ALIAS=ldcSystemPassword}
# enable support for nested groups using the LDAP_MATCHING_RULE_IN_CHAIN operator
ldapRealm.groupSearchEnableMatchingRuleInChain = true
# optional mapping from physical groups to logical application roles
ldapRealm.rolesByGroup = LDN_USERS: user_role, NYK_USERS: user_role, HKG_USERS: user_role, GLOBAL_ADMIN: admin_role
# optional list of roles that are allowed to authenticate. Incase not present all groups are allowed to authenticate (login).
# This changes nothing for url specific permissions that will continue to work as specified in [urls].
ldapRealm.allowedRolesForAuthentication = admin_role,user_role
ldapRealm.permissionsByRole= user_role = *:ToDoItemsJdo:*:*, *:ToDoItem:*:*; admin_role = *
securityManager.sessionManager = $sessionManager
securityManager.realms = $ldapRealm
```

Also instead of specifying systemPassword in clear text in `shiro.ini` administrator can choose to specify the same in "hadoop credential". 
Create a keystore file using the hadoop credential command line:
``` 
hadoop credential create ldapRealm.systemPassword -provider jceks://file/user/zeppelin/conf/zeppelin.jceks
```

Add the following line in the `shiro.ini` file:
``` 
ldapRealm.hadoopSecurityCredentialPath = jceks://file/user/zeppelin/conf/zeppelin.jceks
```
**Caution** due to a bug in LDAPRealm only ```ldapRealm.pagingSize``` results will be fetched from LDAP. In big directory Trees this may cause missing Roles. Try limiting the search Scope using ```ldapRealm.groupSearchBase``` or narrow down the required Groups using ```ldapRealm.groupSearchFilter```

### PAM
[PAM](https://en.wikipedia.org/wiki/Pluggable_authentication_module) authentication support allows the reuse of existing authentication
moduls on the host where Zeppelin is running. On a typical system modules are configured per service for example sshd, passwd, etc. under `/etc/pam.d/`. You can
either reuse one of these services or create your own for Zeppelin. Activiting PAM authentication requires two parameters:
 1. realm: The Shiro realm being used
 2. service: The service configured under `/etc/pam.d/` to be used. The name here needs to be the same as the file name under `/etc/pam.d/`

```
[main]
 pamRealm=org.apache.zeppelin.realm.PamRealm
 pamRealm.service=sshd
```

### Knox SSO
[KnoxSSO](https://knox.apache.org/books/knox-0-13-0/dev-guide.html#KnoxSSO+Integration) provides an abstraction for integrating any number of authentication systems and SSO solutions and enables participating web applications to scale to those solutions more easily. Without the token exchange capabilities offered by KnoxSSO each component UI would need to integrate with each desired solution on its own.

To enable this, apply the following change in `conf/shiro.ini` under `[main]` section.

```
### A sample for configuring Knox JWT Realm
knoxJwtRealm = org.apache.zeppelin.realm.jwt.KnoxJwtRealm
## Domain of Knox SSO
knoxJwtRealm.providerUrl = https://domain.example.com/
## Url for login
knoxJwtRealm.login = gateway/knoxsso/knoxauth/login.html
## Url for logout
knoxJwtRealm.logout = gateway/knoxssout/api/v1/webssout
knoxJwtRealm.redirectParam = originalUrl
knoxJwtRealm.cookieName = hadoop-jwt
knoxJwtRealm.publicKeyPath = /etc/zeppelin/conf/knox-sso.pem
# This is required if KNOX SSO is enabled, to check if "knoxJwtRealm.cookieName" cookie was expired/deleted.  
authc = org.apache.zeppelin.realm.jwt.KnoxAuthenticationFilter
```

### HTTP SPNEGO Authentication
HTTP SPNEGO (Simple and Protected GSS-API NEGOtiation) is the standard way to support Kerberos Ticket based user authentication for Web Services. Based on [Apache Hadoop Auth](https://hadoop.apache.org/docs/current/hadoop-auth/index.html), Zeppelin supports ability to authenticate users by accepting and validating their Kerberos Ticket.

When HTTP SPNEGO Authentication is enabled for Zeppelin, the [Apache Hadoop Groups Mapping](https://hadoop.apache.org/docs/r2.8.0/hadoop-project-dist/hadoop-common/GroupsMapping.html) configuration will used internally to determine group membership of user who is trying to log in. Role-based access permission can be set based on groups as seen by Hadoop.

To enable this, apply the following change in `conf/shiro.ini` under `[main]` section.

```
krbRealm = org.apache.zeppelin.realm.kerberos.KerberosRealm
krbRealm.principal=HTTP/zeppelin.fqdn.domain.com@EXAMPLE.COM
krbRealm.keytab=/etc/security/keytabs/spnego.service.keytab
krbRealm.nameRules=DEFAULT
krbRealm.signatureSecretFile=/etc/security/http_secret
krbRealm.tokenValidity=36000
krbRealm.cookieDomain=domain.com
krbRealm.cookiePath=/
authc = org.apache.zeppelin.realm.kerberos.KerberosAuthenticationFilter
```
For above configuration to work, user need to do some more configurations outside Zeppelin.

1). A valid SPNEGO keytab should be available on the Zeppelin node and should be readable by 'zeppelin' user. If there is a SPNEGO keytab already available (because of other Hadoop service), it can be reused here and no need to generate a new keytab. An example of working SPNEGO keytab could be:
```
$ klist -kt /etc/security/keytabs/spnego.service.keytab
Keytab name: FILE:/etc/security/keytabs/spnego.service.keytab
KVNO Timestamp           Principal
---- ------------------- ------------------------------------------------------
   2 11/26/2018 16:58:38 HTTP/zeppelin.fqdn.domain.com@EXAMPLE.COM
   2 11/26/2018 16:58:38 HTTP/zeppelin.fqdn.domain.com@EXAMPLE.COM
   2 11/26/2018 16:58:38 HTTP/zeppelin.fqdn.domain.com@EXAMPLE.COM
   2 11/26/2018 16:58:38 HTTP/zeppelin.fqdn.domain.com@EXAMPLE.COM
```
and the keytab permission should be: (VERY IMPORTANT to not to set this to 777 or readable by all !!!):
```
$ ls -l /etc/security/keytabs/spnego.service.keytab
-r--r-----. 1 root hadoop 346 Nov 26 16:58 /etc/security/keytabs/spnego.service.keytab
```
Above 'zeppelin' user happens to be member of 'hadoop' group.

2). A secret signature file must be present on Zeppelin node (readable to 'zeppelin' user). This file contains the random binary numbers which is used to sign 'hadoop.auth' cookie, generated during SPNEGO exchange. If such a file is already generated and available on the Zeppelin node, it should be used rather than generating a new file.

Commands to generate a secret signature file (if required):
```
dd if=/dev/urandom of=/etc/security/http_secret bs=1024 count=1
chown hdfs:hadoop /etc/security/http_secret
chmod 440 /etc/security/http_secret
```

## Secure Cookie for Zeppelin Sessions (optional)
Zeppelin can be configured to set `HttpOnly` flag in the session cookie. With this configuration, Zeppelin cookies can 
not be accessed via client side scripts thus preventing majority of Cross-site scripting (XSS) attacks.

To enable secure cookie support via Shiro, add the following lines in `conf/shiro.ini` under `[main]` section, after
defining a `sessionManager`.

```
cookie = org.apache.shiro.web.servlet.SimpleCookie
cookie.name = JSESSIONID
cookie.secure = true
cookie.httpOnly = true
sessionManager.sessionIdCookie = $cookie
```

## Secure your Zeppelin information (optional)
By default, anyone who defined in `[users]` can share **Interpreter Setting**, **Credential** and **Configuration** information in Apache Zeppelin.
Sometimes you might want to hide these information for your use case.
Since Shiro provides **url-based security**, you can hide the information by commenting or uncommenting these below lines in `conf/shiro.ini`.

```
[urls]

/api/interpreter/** = authc, roles[admin]
/api/configurations/** = authc, roles[admin]
/api/credential/** = authc, roles[admin]
```

In this case, only who have `admin` role can see **Interpreter Setting**, **Credential** and **Configuration** information.
If you want to grant this permission to other users, you can change **roles[ ]** as you defined at `[users]` section.

### Apply multiple roles in Shiro configuration
By default, Shiro will allow access to a URL if only user is part of "**all the roles**" defined like this:

```
[urls]

/api/interpreter/** = authc, roles[admin, role1]
```

### Apply multiple roles or user in Shiro configuration
If there is a need that user with "**any of the defined roles or user itself**" should be allowed, then following Shiro configuration can be used:

```
[main]
anyofrolesuser = org.apache.zeppelin.utils.AnyOfRolesUserAuthorizationFilter

[urls]

/api/interpreter/** = authc, anyofrolesuser[admin, user1]
/api/configurations/** = authc, roles[admin]
/api/credential/** = authc, roles[admin]
```

<br/>

> **NOTE :** All of the above configurations are defined in the `conf/shiro.ini` file.


## FAQ

Zeppelin sever is configured as form-based authentication but is behind proxy configured as basic-authentication for example [NGINX](./authentication_nginx.html#http-basic-authentication-using-nginx) and don't want Zeppelin-Server to clear authentication headers. 

> Set `zeppelin.server.authorization.header.clear` to `false` in zeppelin-site.xml

## Other authentication methods

- [HTTP Basic Authentication using NGINX](./authentication_nginx.html)
