## �٣??

```shell
www.baidu.com.
��� .
??�٣ cn edu 
�?�٣ com net ac
߲?�٣ 263 cnnic 163
��?�٣ isd rds albert www

ps �� ������?�٣ʦ���?��󭣬������?�������
```

## DNS���?��

?	**??���**���DNS??٣?����飬?����������٣?��?���������?��������??����������٣?��?������DNS��?Ӯ?���ܣ�줡����٣?��?�?����������������٣?��?����DNS��?Ӯ������?��???��?ʦ��

![image-20191029153157272](C:\Users\86155\AppData\Roaming\Typora\typora-user-images\image-20191029153157272.png)

?	**�������**�� �����??������ݻ��DNS��?Ӯ���?����줡�DNS��?Ӯ�����?���������?������?��?��?��������٣?���۰�� 	��![image-20191029153435101](C:\Users\86155\AppData\Roaming\Typora\typora-user-images\image-20191029153435101.png)

?	�??���٣?��?��?��������?Ӯ��?ϴ?����?����?����????��?�DNS?ϴ??ݻ��RD�ӫ?����1����?��?��?Դ?���?�ϴ�???��????���?����???����������

?	��?Ӯ�DNS?ϴ?������?��������????�������RD�ӫ��1������ӣ�����������٣?��?��߾������????��DNS��?��������?��????�ܣ���?�?��DNS?��?ݻ��RA�ӫ��0��

## ݻ��

```shell
TCP UDP 53 ��UDP?��?������?512�? ����512?����TCP��
/etc/hosts                                   #����٣������� ��??�?����
/etc/named.conf                              #���������
/etc/named.rfc1912.zones                     #��������죨�������죩
/etc/nsswitch.conf                           #?��DNS����?������
/var/named/named.localhost                   #�٣�������
/var/named/named.loopback                    #�����٣������죨��?IP������٣��
named-checkconf /etc/named.conf              #??��������̫��
named-checkzone gong.loopback gong.localhost #??̫��


1.yum install -y bind                        #��?bind
2.vi /etc/named.conf
allow-query { any; };         #��?������??
listen-on port 53 { any; };   #??���IP��
3.vi /etc/named.rfc1912.zones
zone "gong.com" IN {		 #����������ǣ�����?���
        type master;          #�DNS��?��
        file "gong.localhost";#��������٣�������
};

zone "128.13.106.in-addr.arpa" IN {#�����٣������죬���IP
        type master;                   #�DNS��?��
        file "gong.loopback";          #��������٣�������
};
4.	cp /var/named/named.localhost /var/named/gong.localhost
	cp /var/named/named.loopback /var/named/gong.loopback
	chgrp named /var/named/gong.loopback
	chgrp named /var/named/gong.localhost ��ʦ?��
5.vi /var/named/gong.localhost
$TTL 1D                           #DNS?����٤�Ѣ ��?1��
@       IN SOA  @ rname.invalid. (#SOA����?��?
                                        0       ; serial#��֪?���������ʥ1
                                        1D      ; refresh#�����������
                                        1H      ; retry#���?��?���
                                        1W      ; expire#��?���
                                        3H )    ; minimum#?�DNS��?��?�3�??ۯ?��
        NS      @
        A       106.13.128.217
www     A       106.13.128.217
ftp     A       106.13.128.217
mail    A       106.13.128.217
@       MX 10   mail.gong.com
web     CNAME   www.gong.com
6.vi /var/named/gong.loopback
TTL 1D
@       IN SOA  @ rname.invalid. (
                                        0       ; serial
                                        1D      ; refresh
                                        1H      ; retry
                                        1W      ; expire
                                        3H )    ; minimum
        NS      @
        A       106.13.128.217
217     PTR     www.gong.com.
7.vi /etc/resolv.conf #??���DNS��?����

```

## ???��

A          �����?? ??�٣??��IP��

AAAA   ��IPV6���٣���

NS       ����?��?? ?????���?��DNS��?��??���

SOA    ����?�?��???

MX       ��?����???��?�?0-30 30���� ?�?���� ??��?໣�

CNAME��?٣??

PTR     ��PTR??���??���??����A??�����??���� ����IP�����?�٣

## �٣�������

nslookup

```shell
yum install -y bind

[root@instance-0g9xoi12 named]# nslookup
> set q=a	#���???��
> www.gong.com
Server:		106.13.128.217
Address:	106.13.128.217#53

Name:	www.gong.com
Address: 106.13.128.217
```

host

```shell
host -t a www.gong.com     #����٣ 
host -t ptr 106.13.128.217 #�������
```

dig

```shell
dig @106.13.128.217 a www.gong.com	#���DNS��?��?��A????
```

## DNS??��ణ�??г����

```shell
1.vi /var/named/gong.localhost
www	0	A	106.13.128.217
	0	A	106.13.128.218
	
2.vi /etc/named.conf
...
allow-query     { any; };
rrset-order     { order cyclic; };
#rrset-order ��߲???fixed random cyclic
	fixed	#??��?A??����������?��ͳ��?��
	random	#??��?��
	cyclic	#?��??��
...
recursion no; #????��ణ�?����ʦ��ʦ��??���Գ��?��
```

## DNS??���

?	DNSʦ�����?���?��IP�򣣬����DNS��?��������?�������ణ�???�������?��??����??������?���������������򣣨������ణ�

?	��?���������?ʫ����??�������???����ط��??��

```shell
1.vi /etc/named.conf	#�?���������������
#zone "." IN {
#       type hint;
#       file "named.ca";
#};

2.vi /etc/named.rfc1912.zones
acl "shanghai" { 123.56.96.0/24; };    #��?ACL߾��
acl "beijing" { 106.13.128.0/24; };    #��?ACL����

view "bj" {
match-clients { beijing; };
zone "." IN {
       type hint;
       file "named.ca";
};
zone "gong.com" IN {
    type master;
    file "gong_bj.localhost";   #����������
};

zone "128.13.106.in-addr.arpa" IN {
    type master;
    file "gong_bj.loopback";
};
};



view "sh" {
match-clients { shanghai; };
zone "." IN {
       type hint;
       file "named.ca";
};
zone "gong.com" IN {
    type master;
    file "gong_sh.localhost";
};

zone "128.13.106.in-addr.arpa" IN {
    type master;
    file "gong_sh.loopback";
};
};

3.cp gong_bj.loopback  gong_sh.loopback 
  cp gong_bj.localhost gong_sh.localhost
  chown named:named gong_sh*
  vi gong_sh.localhost  #������������
  vi gong_sh.loopback   #������������

4.��?Ӯ??
```

## ?�DNS

?	?�DNS��?������������?�DNS??����٣������£������DNS��?��ʦ�����??�DNS��?�����?�DNS��?��ʦ�?��?DNS��?���?𾡣��?�DNS��?��???DOWN����?�DNS��?������???���ģ�TTL??��

```shell
1.yum install -y bind

2.vi /etc/named.conf
allow-query     { any; };         #��?������??
listen-on port 53 { any; };       #??���IP��

3.vi /etc/named.rfc1912.zones
zone "gong.com" IN {
        type slave;                   #?��??��?��
        file "slaves/gong.localhost"; #???slaves/* ������master߾���٣��������������/var/named/slaves/ �������?masters/* ������������?���?���� 
        masters { 192.168.1.100; };   #���master��
};
zone "1.168.192.in-addr.arpa" IN {
        type slave;
        file "slaves/gong.loopback";
        masters { 192.168.1.100; };
};
```

## DNS+MYSQL

񼣺??�������?mysql������� sql?ϣ���� ��??��mysql�??��