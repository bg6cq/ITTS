# 在 ProFTPd 中配置 LDAP 认证

作者：西北农林科技大学 卞一帆

## 为什么要配置 LDAP 认证

通过 LDAP 方式接入学校的统一身份认证，使得服务器的审计更加准确，便于追踪用户的行为。

## 环境准备

1. Linux 服务器（ CentOS 和 Debian 系均可，本例中使用 CentOS 6.9 ）
2. 安装 ProFTPd （本例中使用编译安装，参考[官方文档](http://www.proftpd.org/docs/howto/Compiling.html)）

## 配置 TLS

本例选择在 `proftpd.conf` 中配置一个 VirtualHost 。如果需要配置到全局，只需要将 TLS 相关指令放到全局配置里。用方括号包括的内容需要根据实际情况修改。

```
<VirtualHost [Your Server IP]>
        ServerName [Your Server Name]
        ServerIdent on "Use Central Authentication System to access this FTP Server"
        DefaultRoot [Path To FTP Root]

        User ftp # change this to adapt this to your server
        Group ftp # change this to adapt this to your server
        # only use LDAP
        AuthOrder mod_ldap.c
        LDAPServer [LDAP Server IP or domain]
        LDAPBindDN [LDAP login user] [LDAP login user password]
        LDAPUsers [Base DN] [User selector]
        # example: LDAP Users "dc=nwafu,dc=edu,dc=cn" "(uid=%u)"
        LDAPSearchScope subtree
        LDAPDefaultUID 15 # change this to adapt this to your server, uid of user ftp
        LDAPDefaultGID 50  # change this to adapt this to your server, gid of group ftp
        LDAPForceDefaultGID on
        LDAPForceDefaultUID on
        RequireValidShell off
        LDAPGenerateHomedir on
        LDAPGenerateHomedirPrefix [FTP Root]
        LDAPForceGeneratedHomedir on
        LDAPGenerateHomedirPrefixNoUsername on

        LDAPLog [Path to LDAP log]

        # Your own configuration goes here
</VirtualHost>
```
