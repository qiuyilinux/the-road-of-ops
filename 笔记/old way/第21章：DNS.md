## æ´Ù£??

```shell
www.baidu.com.
ĞÆæ´ .
??æ´Ù£ cn edu 
ì£?æ´Ù£ com net ac
ß²?æ´Ù£ 263 cnnic 163
ŞÌ?æ´Ù£ isd rds albert www

ps £º Üô÷×Ôõ?æ´Ù£Ê¦ì¤õó?Òıó­£¬ßÓÔÒÔõ?ñşêóìéó­
```

## DNSú°à°?ïï

?	**??ú°à°**£ºî¤DNS??Ù£?ú°à°ñé£¬?á¶ÛÕöÇîÜÜâò¢Ù£?Ü×?Ğïú°à°ÜôÖõ?£¬ı¨ØüîÜ??ÍïíÂãÀë¦Üâò¢Ù£?Ü×?ĞïôğÓÛDNSËÔ?Ó®?ú¼îÜ£¨ì¤¡°Üâò¢Ù£?Ü×?Ğï¡±?ñéãı£©£¬ñşâÍé©Üâò¢Ù£?Ü×?Ğïú¾DNSËÔ?Ó®Ú÷üŞõÌ?îÜ???Íı?Ê¦¡£

![image-20191029153157272](C:\Users\86155\AppData\Roaming\Typora\typora-user-images\image-20191029153157272.png)

?	**ò÷ÓÛú°à°**£º îÜá¶êó??ÍïíÂîïİ»ãÀDNSËÔ?Ó®í»Ğù?ú¼£¨ì¤¡°DNSËÔ?Ó®¡±í»Ğù?ñéãı£©¡£î¤?Ëìñıìé?ğë?ö¦?óúéÄò÷ÓÛÙ£?ú°à°Û°ãÒ 	¡£![image-20191029153435101](C:\Users\86155\AppData\Roaming\Typora\typora-user-images\image-20191029153435101.png)

?	î¤??Üâò¢Ù£?Ü×?Ğï?£¬åıÍıËÔ?Ó®îÜ?Ï´?Ùşñé?êóãé?ŞÅéÄ????£¬?î¤DNS?Ï´??İ»îÜRDí®Ó«?êóöÇ1¡£ßÓ?éÍ?¡°?Ô´?êóñ«?é©Ï´ä²???ú¼????£¬ä²?æÔÜô???ÍïíÂÖõ¡±¡£

?	ËÔ?Ó®î¤DNS?Ï´?Ùşñéãé?ŞÅéÄîÜãÀ????£¨å¥ö¦ãÀRDí®Ó«öÇ1Öõ£©£¬Ó£î¤á¶ÛÕöÇîÜÜâò¢Ù£?Ü×?Ğïß¾ãÀĞ×éÄ????£¨DNSÜ×?ĞïìéÚõÙù?ò¨ò¥????îÜ£©£¬?î¤?ÓÍDNS?Ùş?İ»îÜRAí®Ó«öÇ0¡£

## İ»ßş

```shell
TCP UDP 53 £¨UDP?Ùş?ÓøõÌÓŞ?512í®? ÓŞéÍ512?ŞÅéÄTCP£©
/etc/hosts                                   #Üâò¢æ´Ù£ú°à°ÙşËì Ùù??à»?õÌÍÔ
/etc/named.conf                              #ñ«ÛÕöÇÙşËì
/etc/named.rfc1912.zones                     #æ´ÛÕöÇÙşËì£¨İÂùÜÙşËì£©
/etc/nsswitch.conf                           #?ïÚDNSßæüù?ßíÙşËì
/var/named/named.localhost                   #æ´Ù£ú°à°ÙşËì
/var/named/named.loopback                    #Úãú¾æ´Ù£ú°à°ÙşËì£¨÷×?IPò¢ò£ú°à°æ´Ù££©
named-checkconf /etc/named.conf              #??ÛÕöÇÙşËìÌ«ãÒ
named-checkzone gong.loopback gong.localhost #??Ì«ãÒ


1.yum install -y bind                        #äÌ?bind
2.vi /etc/named.conf
allow-query { any; };         #ëÃ?á¶êóñ«Ïõ??
listen-on port 53 { any; };   #??á¶êóIPò¢ò£
3.vi /etc/named.rfc1912.zones
zone "gong.com" IN {		 #ïáú¾ú°à°ÛÕöÇ£¬ò¦ïÒñ¼?îÜæ´
        type master;          #ñ«DNSÜ×?Ğï
        file "gong.localhost";#ò¦ïÒïáú¾æ´Ù£ú°à°ÙşËì
};

zone "128.13.106.in-addr.arpa" IN {#Úãú¾æ´Ù£ú°à°ÙşËì£¬ò¦ïÒIP
        type master;                   #ñ«DNSÜ×?Ğï
        file "gong.loopback";          #ò¦ïÒÚãú¾æ´Ù£ú°à°ÙşËì
};
4.	cp /var/named/named.localhost /var/named/gong.localhost
	cp /var/named/named.loopback /var/named/gong.loopback
	chgrp named /var/named/gong.loopback
	chgrp named /var/named/gong.localhost £¨Ê¦?£©
5.vi /var/named/gong.localhost
$TTL 1D                           #DNS?ğíßæÙ¤ñ²Ñ¢ Ùù?1ô¸
@       IN SOA  @ rname.invalid. (#SOAÑÃã·â£?Ïõ?
                                        0       ; serial#ßíÖª?Øßáìãæìéó­Ê¥1
                                        1D      ; refresh#ìéô¸áìãæìéó­
                                        1H      ; retry#ìéá³?ñì?ìéó­
                                        1W      ; expire#ñì?ìéñ²
                                        3H )    ; minimum#?ğ¾DNSÜ×?Ğï?ò¥3á³??Û¯?Ëß
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
7.vi /etc/resolv.conf #??ò¦ïÒDNSÜ×?Ğïò¢ò£

```

## ???úş

A          £ºñ«Ïõ?? ??æ´Ù£??îÜIPò¢ò£

AAAA   £ºIPV6îÜæ´Ù£ú°à°

NS       £ºÜ×?Ğï?? ?????æ´êó?ŞÁDNSÜ×?Ğï??ú°à°

SOA    £ºĞì?â£?Ïõ???

MX       £º?ËìÎß???£¨?à»?0-30 30õÌÍÔ ?à»?êÆÍÔ ??êÆ?à»£©

CNAME£º?Ù£??

PTR     £ºPTR??å¥ù¬??ú°à°??£¬ûúA??ãÀæ½ú¾??íÂéÄ ãÀ÷êIPò¢ò£ú°à°?æ´Ù£

## æ´Ù£ú°à°ÍïÎı

nslookup

```shell
yum install -y bind

[root@instance-0g9xoi12 named]# nslookup
> set q=a	#ò¦ïÒ???úş
> www.gong.com
Server:		106.13.128.217
Address:	106.13.128.217#53

Name:	www.gong.com
Address: 106.13.128.217
```

host

```shell
host -t a www.gong.com     #ú°à°æ´Ù£ 
host -t ptr 106.13.128.217 #Úãú¾ú°à°
```

dig

```shell
dig @106.13.128.217 a www.gong.com	#ò¦ïÒDNSÜ×?Ğï?ú¼A????
```

## DNS??ú°à°£¨??Ğ³û¬£©

```shell
1.vi /var/named/gong.localhost
www	0	A	106.13.128.217
	0	A	106.13.128.218
	
2.vi /etc/named.conf
...
allow-query     { any; };
rrset-order     { order cyclic; };
#rrset-order ò¨ò¥ß²???fixed random cyclic
	fixed	#??Òı?A??äÎÛÕöÇÙşËì?ßíÍ³ïÒ?õó
	random	#??Ïõ?õó
	cyclic	#?âà??õó
...
recursion no; #????ú°à°£¨?ìéÜÆÊ¦êóÊ¦Ùé??Üôò±Ô³éÄ?£©
```

## DNS??ú°à°

?	DNSÊ¦ì¤ĞÆËß?ê¹ËÔ?îÜIPò¢ò££¬ëîËßDNSÜ×?ĞïîÜò¢ò£øú?ú¼òªÒöú°à°£¬???éÍÜô÷×ò¢?ûä??Üô÷×??ßÂîÜéÄ?ò¦ú¾ÓğÜôéÄîÜú°à°ò¢ò££¨ö¦ĞÎú°à°£©

?	ú°?Öõë¦éÍÜôÔÒ?Ê«ÜôÔÒ??ßÂÜôÔÒò¢???áÜÓøØ·îÜ??¡£

```shell
1.vi /etc/named.conf	#ñ¼?ñ«ÛÕöÇÙşËìñ«æ´ãáãÓ
#zone "." IN {
#       type hint;
#       file "named.ca";
#};

2.vi /etc/named.rfc1912.zones
acl "shanghai" { 123.56.96.0/24; };    #ïÒ?ACLß¾ú­
acl "beijing" { 106.13.128.0/24; };    #ïÒ?ACLİÁÌÈ

view "bj" {
match-clients { beijing; };
zone "." IN {
       type hint;
       file "named.ca";
};
zone "gong.com" IN {
    type master;
    file "gong_bj.localhost";   #ò¦ïÒú°à°ÙşËì
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
  vi gong_sh.localhost  #áóËÇÛÕöÇÙşËì
  vi gong_sh.loopback   #áóËÇÛÕöÇÙşËì

4.ËÔ?Ó®??
```

## ?ğ¾DNS

?	?ñ«DNSÜ×?Ğï÷µÏõñıı¨£¬?ğ¾DNS??ğ«Íêæ´Ù£ú°à°ÍïíÂ£¬ìé÷»ñ«DNSÜ×?ĞïÊ¦ì¤êóÒı??ğ¾DNSÜ×?Ğï¡£ìé÷»?ğ¾DNSÜ×?ĞïÊ¦ì¤?Òı?DNSÜ×?Ğïñ®?ğ¾¡£÷Î?ñ«DNSÜ×?Ğï???DOWNÏõ£¬?ğ¾DNSÜ×?ĞïÜôÒö???ŞÅéÄ£¨TTL??£©

```shell
1.yum install -y bind

2.vi /etc/named.conf
allow-query     { any; };         #ëÃ?á¶êóñ«Ïõ??
listen-on port 53 { any; };       #??á¶êóIPò¢ò£

3.vi /etc/named.rfc1912.zones
zone "gong.com" IN {
        type slave;                   #?úş??Ü×?Ğï
        file "slaves/gong.localhost"; #???slaves/* ãÀÔÒÜÆmasterß¾îÜæ´Ù£ú°à°ÙşËìÓğÜâò¢îÜ/var/named/slaves/ òÁïÈò¦ïÒ?masters/* ãÀÜôÔÒÜÆòÁïÈ?ñ«Ü×?ĞïîÜ 
        masters { 192.168.1.100; };   #ò¦ïÒmasterò¢ò£
};
zone "1.168.192.in-addr.arpa" IN {
        type slave;
        file "slaves/gong.loopback";
        masters { 192.168.1.100; };
};
```

## DNS+MYSQL

ñ¼£º??ÛÕöÇâÍé©?mysqlä¢ìıÖõú° sql?Ï£âÙãú Ôõ??èÇmysqlî¢??¡£