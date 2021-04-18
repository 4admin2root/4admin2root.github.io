---
layout:     post
title:      各类web服务器httpaccesslog获取用户和代理ip
date:       2014-01-16
---
[IIS、apache、nginx日志中获取访客真实IP的解决方案](http://bbs.jiasule.com/thread-3957-1-1.html)

[http://bbs.jiasule.com/thread-3957-1-1.html](http://bbs.jiasule.com/thread-3957-1-1.html)

1. [SETTINGS] 
1. HEADER=X-Forwarded-For



1. #LogFormat "%h %l %u %t \"%r\" %>s %b \"%{Referer}i\" \"%{User-Agent}i\"" combined
1. #LogFormat "%h %l %u %t \"%r\" %>s %b" common
1. #<IfModule logio_module>
1. #   LogFormat "%h %l %u %t \"%r\" %>s %b \"%{Referer}i\" \"%{User-Agent}i\" %I %O" combinedio
1. #</IfModule>
1. LogFormat "%{X-Forwarded-For}i %l %u %t \"%r\" %>s %b \"%{Referer}i\" \"%{User-Agent}i\" CacheIP:%h" combined
1. LogFormat "%{X-Forwarded-For }i %l %u %t \"%r\" %>s %b CacheIP:%h" common
1. <IfModule logio_module>
1.    LogFormat "%{X-Forwarded-For}i %l %u %t \"%r\" %>s %b \"%{Referer}i\" \"%{User-Agent}i\" %I %O CacheIP:%h" combinedio
1. </IfModule>



1. log_format  main  '$http_x_forwarded_for - $remote_user [$time_local] "$request" '
1.                  '$status $body_bytes_sent "$http_referer" '
1.                  '"$http_user_agent" "CacheIP:$remote_addr" ';



[F5XForwardedFor2008.zip](http://bbs.jiasule.com/forum.php?mod=attachment&aid=MTEwNnxkNGViODY3NXwxMzg5ODQ5ODQzfDB8Mzk1Nw%3D%3D)

44.51 KB, 下载次数: 114

 

# [tomcat记录X-Forwarded-For字段中的远程IP](http://www.cnblogs.com/anic/archive/2012/12/13/2817126.html)

[http://www.cnblogs.com/anic/archive/2012/12/13/2817126.html](http://www.cnblogs.com/anic/archive/2012/12/13/2817126.html)

## F5转包时将请求IP记录到X-Forwarded-For字段

最近在做某个系统的安全加固的时候，发现tomcat记录的日志全部来自于F5转发的IP地址，不能获取到请求的真实IP。

经过抓包分析，F5在转发请求包的时候会将来源IP记录在HTTP包header中的X-Forwarded-For字段，如下图是用wireshark（[www.wireshark.org/download.html](http://www.wireshark.org/download.html)）截取的包。

[<img style="margin: 0px; padding: 0px; border: 0px; display: inline;" title="image" src="http://images.cnblogs.com/cnblogs_com/anic/201212/201212132150156709.png" alt="image" width="761" height="335" border="0" />](http://images.cnblogs.com/cnblogs_com/anic/201212/201212132150113235.png)

 

而Tomcat是在tomcat/conf/server.xml中配置日志选项的：

 

# Tomcat如何记录日志

禁用<AccessLogValue>，则对应logs目录下不会生成localhost_access_log.txt

[<img style="margin: 0px; padding: 0px; border: 0px; display: inline;" title="clip_image002" src="http://images.cnblogs.com/cnblogs_com/anic/201212/201212132150199344.jpg" alt="clip_image002" width="561" height="103" border="0" />](http://images.cnblogs.com/cnblogs_com/anic/201212/201212132150155804.jpg)

Logs下没有localhost_access_log.txt见：

[<img style="margin: 0px; padding: 0px; border: 0px; display: inline;" title="clip_image004" src="http://images.cnblogs.com/cnblogs_com/anic/201212/201212132150207817.jpg" alt="clip_image004" width="431" height="143" border="0" />](http://images.cnblogs.com/cnblogs_com/anic/201212/201212132150196247.jpg)

 

当开启<AccessLogValue>，则会有localhost_access_log.txt日志记录请求来源

[<img style="margin: 0px; padding: 0px; border: 0px; display: inline;" title="clip_image002[6]" src="http://images.cnblogs.com/cnblogs_com/anic/201212/201212132150214130.jpg" alt="clip_image002[6]" width="558" height="50" border="0" />](http://images.cnblogs.com/cnblogs_com/anic/201212/201212132150205276.jpg)

当配置中的pattern=common时，对应的日志是如下，无论正常请求和非法请求都会记录。

[<img style="margin: 0px; padding: 0px; border: 0px; display: inline;" title="clip_image004[5]" src="http://images.cnblogs.com/cnblogs_com/anic/201212/201212132150263045.jpg" alt="clip_image004[5]" width="529" height="83" border="0" />](http://images.cnblogs.com/cnblogs_com/anic/201212/2012121321502294.jpg)

 

## 记录远程IP的两个方法

### 方案1

修改pattern为pattern='%{X-Forwarded-For}i %h %l %u %t "%r" %s %b'，则会记录headers头中的X-Forwarded-For信息

[<img style="margin: 0px; padding: 0px; border: 0px; display: inline;" title="clip_image002[12]" src="http://images.cnblogs.com/cnblogs_com/anic/201212/201212132150279599.jpg" alt="clip_image002[12]" width="581" height="50" border="0" />](http://images.cnblogs.com/cnblogs_com/anic/201212/201212132150268552.jpg)

 

如果没有则记录为空，如下

[<img style="margin: 0px; padding: 0px; border: 0px; display: inline;" title="clip_image002[8]" src="http://images.cnblogs.com/cnblogs_com/anic/201212/201212132150282781.jpg" alt="clip_image002[8]" width="649" height="109" border="0" />](http://images.cnblogs.com/cnblogs_com/anic/201212/201212132150274516.jpg)

验证：用fiddle构造有X-Forwarded-For的请求包

原始请求

[<img style="margin: 0px; padding: 0px; border: 0px; display: inline;" title="clip_image004[7]" src="http://images.cnblogs.com/cnblogs_com/anic/201212/201212132150319693.jpg" alt="clip_image004[7]" width="571" height="285" border="0" />](http://images.cnblogs.com/cnblogs_com/anic/201212/201212132150309825.jpg)

构造X-Forwarded-For字段

[<img style="margin: 0px; padding: 0px; border: 0px; display: inline;" title="clip_image006" src="http://images.cnblogs.com/cnblogs_com/anic/201212/201212132150333332.jpg" alt="clip_image006" width="573" height="286" border="0" />](http://images.cnblogs.com/cnblogs_com/anic/201212/201212132150329561.jpg)

服务器localhost_access_log.txt

[<img style="margin: 0px; padding: 0px; border: 0px; display: inline;" title="clip_image008" src="http://images.cnblogs.com/cnblogs_com/anic/201212/201212132150342710.jpg" alt="clip_image008" width="590" height="28" border="0" />](http://images.cnblogs.com/cnblogs_com/anic/201212/201212132150346887.jpg)

 

### 方案2

不修改pattern，增加RemoteIpValue配置

[<img style="margin: 0px; padding: 0px; border: 0px; display: inline;" title="clip_image002[10]" src="http://images.cnblogs.com/cnblogs_com/anic/201212/201212132150352611.jpg" alt="clip_image002[10]" width="513" height="85" border="0" />](http://images.cnblogs.com/cnblogs_com/anic/201212/201212132150359612.jpg)

前两个是记录X-Forworded-For，后面是没有X-Forwarded-For，说明是有效的

[<img style="margin: 0px; padding: 0px; border: 0px; display: inline;" title="clip_image004[9]" src="http://images.cnblogs.com/cnblogs_com/anic/201212/201212132150381475.jpg" alt="clip_image004[9]" width="544" height="148" border="0" />](http://images.cnblogs.com/cnblogs_com/anic/201212/201212132150363559.jpg)

以上的这些都参考了相关文档，下面这些才是实验。

 

## 方案比较

### 比较1：如果有X-Forwarded-For头，但是内容为空。

方案1显示空和转发IP

方案2显示直接来源IP

 

### 比较2：多个X-Forwarded-For字段

例如一个包里面有两个X-Forwarded-For字段

X-Forwarded-For：59.66.156.24

X-Forwarded-For：59.66.156.111

 

方案1日志为（记录第一个）：59.66.156.24 192.168.1.58 - - [12/Dec/2012:22:56:54 +0800] "GET /docs/ HTTP/1.1" 304 -

方案2日志为（记录第一个）：59.66.156.24 - - [12/Dec/2012:22:50:16 +0800] "GET /docs/ HTTP/1.1" 304 

 

### 比较3：X-Forwarded-For有多个IP

例如：X-Forwarded-For：59.66.156.24,59.66.156.111

 

方案1日志为显示完整X-Forwarded-For字段信息：

59.66.156.24,59.66.156.111 192.168.1.58 - - [12/Dec/2012:22:58:01 +0800] "GET /docs/ HTTP/1.1" 304 -

方案2日志记录为（记录最后一个IP）：

59.66.156.111 - - [12/Dec/2012:22:53:51 +0800] "GET /docs/ HTTP/1.1" 304 -

 

### 结论

方案1真实反映X-Forwarded-For字段（无论有无IP，有多少个IP），并且以header头按顺序读取，显示读取到第一个X-Forwarded-For。

方案2试图将IP进行来源替换，如果有X-Forwarded-For字段，则显示最后一个IP，否则显示直接IP

 

因此从日志记录来看，方案1的信息量更为大。

 

## 遗留问题

如果有人伪造了多个X-Forwarded-For字段，而F5转包时，到底会将请求IP加到那个X-Forwarded-For字段呢？我只能寄希望于他会在第一个X-Forwarded-For中添加这个信息，否则tomcat就无法记录了。

 

**weblogic****使用扩展日志格式**

[http://docs.oracle.com/cd/E15523_01/web.1111/e13701/web_server.htm#i1073749](http://docs.oracle.com/cd/E15523_01/web.1111/e13701/web_server.htm#i1073749)

**Setting Up HTTP Access Logs**

WebLogic Server can keep a log of all HTTP transactions in a text file, in either **common log format** or **extended log format**. Common log format is the default. Extended log format allows you to customize the information that is recorded. You can set the attributes that define the behavior of HTTP access logs for each server instance or for each virtual host that you define.

To set up HTTP logging for a server or a virtual host, refer to the following topics in the **Oracle Fusion Middleware Oracle WebLogic Server Administration Console Help**:

- ["Enabling and Configuring HTTP Access Logs"](http://docs.oracle.com/cd/E15523_01/apirefs.1111/e13952/taskhelp/logging/EnableAndConfigureHTTPLogs.html)
- ["Specifying HTTP Log File Settings for a Virtual Host"](http://docs.oracle.com/cd/E15523_01/apirefs.1111/e13952/taskhelp/virtual_hosts/ConfigureVirtualHostLogging.html)


**Log Rotation**

You can rotate the log file based on either the size of the file or after a specified amount of time has passed. When either criterion is met, the current access log file is closed and a new access log file is started. If you do not configure log rotation, the HTTP access log file grows indefinitely. You can configure the name of the access log file to include a time and date stamp that indicates when the file was rotated. If you do not configure a time stamp, each rotated file name includes a numeric portion that is incremented upon each rotation. Separate HTTP access logs are kept for each Virtual Host you have defined.

**Common Log Format**

The default format for logged HTTP information is the common log format (see [http://www.w3.org/Daemon/User/Config/Logging.html#common-logfile-format](http://www.w3.org/Daemon/User/Config/Logging.html#common-logfile-format)). This standard format follows the pattern:

host RFC931 auth_user [day/month/year:hour:minute:second **UTC_offset**] "request" status bytes

where:

**host**

Either the DNS name or the IP number of the remote client

**RFC931**

Any information returned by IDENTD for the remote client; WebLogic Server does not support user identification

**auth_user**

If the remote client user sent a userid for authentication, the user name; otherwise "-"

**day/month/year:hour:minute:second UTC_offset**

Day, calendar month, year and time of day (24-hour format) with the hours difference between local time and GMT, enclosed in square brackets

**"request"**

First line of the HTTP request submitted by the remote client enclosed in double quotes

**status**

HTTP status code returned by the server, if available; otherwise "-"

**bytes**

Number of bytes listed as the content-length in the HTTP header, not including the HTTP header, if known; otherwise "-"

**Setting Up HTTP Access Logs by Using Extended Log Format**

WebLogic Server also supports extended log file format, version 1.0, an emerging standard defined by the draft specification from the W3C at [http://www.w3.org/TR/WD-logfile.html](http://www.w3.org/TR/WD-logfile.html). The current definitive reference is on the W3C Technical Reports and Publications page at [http://www.w3.org/TR/](http://www.w3.org/TR/).

The extended log format allows you to specify the type and order of information recorded about each HTTP communication. To enable this format, set the Format attribute on the HTTP tab in the Administration Console to Extended. (See [Creating Custom Field Identifiers](http://docs.oracle.com/cd/E15523_01/web.1111/e13701/web_server.htm#i1059518)).

You specify what information should be recorded in the log file with directives, included in the actual log file itself. A directive begins on a new line and starts with a # sign. If the log file does not exist, a new log file is created with default directives. However, if the log file already exists when the server starts, it must contain legal directives at the head of the file.

**Creating the Fields Directive**

The first line of your log file must contain a directive stating the version number of the log file format. You must also include a Fields directive near the beginning of the file:

#Version: 1.0

#Fields: xxxx xxxx xxxx ...

Where each xxxx describes the data fields to be recorded. Field types are specified as either simple identifiers, or may take a prefix-identifier format, as defined in the W3C specification. Here is an example:

#Fields: date time cs-method cs-uri

This identifier instructs the server to record the date and time of the transaction, the request method that the client used, and the URI of the request for each HTTP access. Each field is separated by white space, and each record is written to a new line, appended to the log file.

**Note:**

The #Fields directive must be followed by a new line in the log file, so that the first log message is not appended to the same line.

**Supported Field identifiers**

The following identifiers are supported, and do not require a prefix.

**date**

Date at which transaction completed, field has type <date>, as defined in the W3C specification.

**time**

Time at which transaction completed, field has type <time>, as defined in the W3C specification.

**time-taken**

Time taken for transaction to complete in seconds, field has type <fixed>, as defined in the W3C specification.

**bytes**

Number of bytes transferred, field has type <integer>.

Note that the cached field defined in the W3C specification is not supported in WebLogic Server.

The following identifiers require prefixes, and cannot be used alone. The supported prefix combinations are explained individually.

**IP address related fields:**

These fields these fields give the IP address and port of either the requesting client, or the responding server. These fields have type <address>, as defined in the W3C specification. The supported prefixes are:

**c-ip**

The IP address of the client.

**s-ip**

The IP address of the server.

**DNS related fields**

These fields give the domain names of the client or the server and have type <name>, as defined in the W3C specification. The supported prefixes are:

**c-dns**

The domain name of the requesting client.

**s-dns**

The domain name of the requested server.

**sc-status**

Status code of the response, for example (404) indicating a ""File not found" status. This field has type <integer>, as defined in the W3C specification.

**sc-comment**

The comment returned with status code, for instance "File not found". This field has type <text>.

**cs-method**

The request method, for example GET or POST. This field has type <name>, as defined in the W3C specification.

**cs-uri**

The full requested URI. This field has type <uri>, as defined in the W3C specification.

**cs-uri-stem**

Only the stem portion of URI (omitting query). This field has type <uri>, as defined in the W3C specification.

**cs-uri-query**

Only the query portion of the URI. This field has type <uri>, as defined in the W3C specification.

**Creating Custom Field Identifiers**

You can also create user-defined fields for inclusion in an HTTP access log file that uses the extended log format. To create a custom field you identify the field in the ELF log file using the Fields directive and then you create a matching Java class that generates the desired output. You can create a separate Java class for each field, or the Java class can output multiple fields. For a sample of the Java source for such a class, see [Java Class for Creating a Custom ELF Field](http://docs.oracle.com/cd/E15523_01/web.1111/e13701/web_server.htm#i1071418).

To create a custom field:

1. Include the field name in the Fields directive, using the form:


1. x-myCustomField.


Where **myCustomField** is a fully-qualified class name.

For more information on the Fields directive, see [Creating the Fields Directive](http://docs.oracle.com/cd/E15523_01/web.1111/e13701/web_server.htm#i1059474).

1. Create a Java class with the same fully-qualified class name as the custom field you defined with the Fields directive (for example myCustomField). This class defines the information you want logged in your custom field. The Java class must implement the following interface:


1. weblogic.servlet.logging.CustomELFLogger


In your Java class, you must implement the logField() method, which takes a HttpAccountingInfo object and FormatStringBuffer object as its arguments:

- Use the HttpAccountingInfo object to access HTTP request and response data that you can output in your custom field. Getter methods are provided to access this information. For a complete listing of these get methods, see [Get Methods of the HttpAccountingInfo Object](http://docs.oracle.com/cd/E15523_01/web.1111/e13701/web_server.htm#i1059551).
- Use the FormatStringBuffer class to create the contents of your custom field. Methods are provided to create suitable output.


1. Compile the Java class and add the class to the CLASSPATH statement used to start WebLogic Server. You will probably need to modify the CLASSPATH statements in the scripts that you use to start WebLogic Server.


**Note:**

Do not place this class inside of a Web application or Enterprise application in exploded or jar format.

1. Configure WebLogic Server to use the extended log format. For more information, see [Setting Up HTTP Access Logs by Using Extended Log Format](http://docs.oracle.com/cd/E15523_01/web.1111/e13701/web_server.htm#i1059459).


**Notes:**

When writing the Java class that defines your custom field, do not execute any code that is likely to slow down the system (For instance, accessing a DBMS or executing significant I/O or networking calls.) Remember, an HTTP access log file entry is created for **every** HTTP request.

**Note:**

If you want to output more than one field, delimit the fields with a tab character. For more information on delimiting fields and other ELF formatting issues, see "Extended Log Format" at [http://www.w3.org/TR/WD-logfile-960221.html](http://www.w3.org/TR/WD-logfile-960221.html).

**Get Methods of the HttpAccountingInfo Object**

The following methods return various data regarding the HTTP request. These methods are similar to various methods of javax.servlet.ServletRequest,javax.servlet.http.Http.ServletRequest, and javax.servlet.http.HttpServletResponse.

The Javadoc for these interfaces is at the following URLs:

- [http://java.sun.com/products/servlet/2.3/javadoc/javax/servlet/ServletRequest.html](http://java.sun.com/products/servlet/2.3/javadoc/javax/servlet/ServletRequest.html)
- [http://java.sun.com/products/servlet/2.3/javadoc/javax/servlet/ServletResponse.html](http://java.sun.com/products/servlet/2.3/javadoc/javax/servlet/ServletResponse.html)
- [http://java.sun.com/products/servlet/2.3/javadoc/javax/servlet/http/HttpServletRequest.html](http://java.sun.com/products/servlet/2.3/javadoc/javax/servlet/http/HttpServletRequest.html)


For details on these methods see the corresponding methods in the Java interfaces listed in the following table, or refer to the specific information contained in the table.

****Table 5-2 Getter Methods of HttpAccountingInfo****

<td valign="bottom">**HttpAccountingInfo Methods**</td><td valign="bottom">**Method Information**</td>
|------
<td valign="top">Object getAttribute(String name);</td><td valign="top">javax.servlet.ServletRequest</td>

javax.servlet.ServletRequest
<td valign="top">Enumeration getAttributeNames();</td><td valign="top">javax.servlet.ServletRequest</td>

javax.servlet.ServletRequest
<td valign="top">String getCharacterEncoding();</td><td valign="top">javax.servlet.ServletRequest</td>

javax.servlet.ServletRequest
<td valign="top">int getResponseContentLength();</td><td valign="top">javax.servlet.ServletResponse.setContentLength()This method gets the content length of the response, as set with the setContentLength()method.</td>

javax.servlet.ServletResponse.setContentLength()
<td valign="top">String getContentType();</td><td valign="top">javax.servlet.ServletRequest</td>

javax.servlet.ServletRequest
<td valign="top">Locale getLocale();</td><td valign="top">javax.servlet.ServletRequest</td>

javax.servlet.ServletRequest
<td valign="top">Enumeration getLocales();</td><td valign="top">javax.servlet.ServletRequest</td>

javax.servlet.ServletRequest
<td valign="top">String getParameter(String name);</td><td valign="top">javax.servlet.ServletRequest</td>

javax.servlet.ServletRequest
<td valign="top">Enumeration getParameterNames();</td><td valign="top">javax.servlet.ServletRequest</td>

javax.servlet.ServletRequest
<td valign="top">String[] getParameterValues(String name);</td><td valign="top">javax.servlet.ServletRequest</td>

javax.servlet.ServletRequest
<td valign="top">String getProtocol();</td><td valign="top">javax.servlet.ServletRequest</td>

javax.servlet.ServletRequest
<td valign="top">String getRemoteAddr();</td><td valign="top">javax.servlet.ServletRequest</td>

javax.servlet.ServletRequest
<td valign="top">String getRemoteHost();</td><td valign="top">javax.servlet.ServletRequest</td>

javax.servlet.ServletRequest
<td valign="top">String getScheme();</td><td valign="top">javax.servlet.ServletRequest</td>

javax.servlet.ServletRequest
<td valign="top">String getServerName();</td><td valign="top">javax.servlet.ServletRequest</td>

javax.servlet.ServletRequest
<td valign="top">int getServerPort();</td><td valign="top">javax.servlet.ServletRequest</td>

javax.servlet.ServletRequest
<td valign="top">boolean isSecure();</td><td valign="top">javax.servlet.ServletRequest</td>

javax.servlet.ServletRequest
<td valign="top">String getAuthType();</td><td valign="top">javax.servlet.http.HttpServletRequest</td>

javax.servlet.http.HttpServletRequest
<td valign="top">String getContextPath();</td><td valign="top">javax.servlet.http.HttpServletRequest</td>

javax.servlet.http.HttpServletRequest
<td valign="top">Cookie[] getCookies();</td><td valign="top">javax.servlet.http.HttpServletRequest</td>

javax.servlet.http.HttpServletRequest
<td valign="top">long getDateHeader(String name);</td><td valign="top">javax.servlet.http.HttpServletRequest</td>

javax.servlet.http.HttpServletRequest
<td valign="top">String getHeader(String name);</td><td valign="top">javax.servlet.http.HttpServletRequest</td>

javax.servlet.http.HttpServletRequest
<td valign="top">Enumeration getHeaderNames();</td><td valign="top">javax.servlet.http.HttpServletRequest</td>

javax.servlet.http.HttpServletRequest
<td valign="top">Enumeration getHeaders(String name);</td><td valign="top">javax.servlet.http.HttpServletRequest</td>

javax.servlet.http.HttpServletRequest
<td valign="top">int getIntHeader(String name);</td><td valign="top">javax.servlet.http.HttpServletRequest</td>

javax.servlet.http.HttpServletRequest
<td valign="top">String getMethod();</td><td valign="top">javax.servlet.http.HttpServletRequest</td>

javax.servlet.http.HttpServletRequest
<td valign="top">String getPathInfo();</td><td valign="top">javax.servlet.http.HttpServletRequest</td>

javax.servlet.http.HttpServletRequest
<td valign="top">String getPathTranslated();</td><td valign="top">javax.servlet.http.HttpServletRequest</td>

javax.servlet.http.HttpServletRequest
<td valign="top">String getQueryString();</td><td valign="top">javax.servlet.http.HttpServletRequest</td>

javax.servlet.http.HttpServletRequest
<td valign="top">String getRemoteUser();</td><td valign="top">javax.servlet.http.HttpServletRequest</td>

javax.servlet.http.HttpServletRequest
<td valign="top">String getRequestURI();</td><td valign="top">javax.servlet.http.HttpServletRequest</td>

javax.servlet.http.HttpServletRequest
<td valign="top">String getRequestedSessionId();</td><td valign="top">javax.servlet.http.HttpServletRequest</td>

javax.servlet.http.HttpServletRequest
<td valign="top">String getServletPath();</td><td valign="top">javax.servlet.http.HttpServletRequest</td>

javax.servlet.http.HttpServletRequest
<td valign="top">Principal getUserPrincipal();</td><td valign="top">javax.servlet.http.HttpServletRequest</td>

javax.servlet.http.HttpServletRequest
<td valign="top">boolean isRequestedSessionIdFromCookie();</td><td valign="top">javax.servlet.http.HttpServletRequest</td>

javax.servlet.http.HttpServletRequest
<td valign="top">boolean isRequestedSessionIdFromURL();</td><td valign="top">javax.servlet.http.HttpServletRequest</td>

javax.servlet.http.HttpServletRequest
<td valign="top">boolean isRequestedSessionIdFromUrl();</td><td valign="top">javax.servlet.http.HttpServletRequest</td>

javax.servlet.http.HttpServletRequest
<td valign="top">boolean isRequestedSessionIdValid();</td><td valign="top">javax.servlet.http.HttpServletRequest</td>

javax.servlet.http.HttpServletRequest
<td valign="top">byte[] getURIAsBytes();</td><td valign="top">Returns the URI of the HTTP request as byte array, for example: If GET /index.html HTTP/1.0 is the first line of an HTTP Request, /index.html is returned as an array of bytes.</td>

Returns the URI of the HTTP request as byte array, for example: If GET /index.html HTTP/1.0 is the first line of an HTTP Request, /index.html is returned as an array of bytes.
<td valign="top">long getInvokeTime();</td><td valign="top">Returns the length of time it took for the service method of a servlet to write data back to the client.</td>

Returns the length of time it took for the service method of a servlet to write data back to the client.
<td valign="top">int getResponseStatusCode();</td><td valign="top">javax.servlet.http.HttpServletResponse</td>

javax.servlet.http.HttpServletResponse
<td valign="top">String getResponseHeader(String name);</td><td valign="top">javax.servlet.http.HttpServletResponse</td>

javax.servlet.http.HttpServletResponse

 

****Example 5-1 Java Class for Creating a Custom ELF Field****

import weblogic.servlet.logging.CustomELFLogger;

import weblogic.servlet.logging.FormatStringBuffer;

import weblogic.servlet.logging.HttpAccountingInfo;

/* This example outputs the User-Agent field into a

 custom field called MyCustomField

*/

public class MyCustomField implements CustomELFLogger{

public void logField(HttpAccountingInfo metrics,

  FormatStringBuffer buff) {

  buff.appendValueOrDash(metrics.getHeader("User-Agent"));

  }

}
