## 1.curl url

作用：获取界面内容或者接口响应。

示例：

![1570861922371](C:\Users\86155\Desktop\1570861922371.png)

## 2.curl -i url

作用：获取请求页面或者接口的请求头信息，可以通过HTTP状态码判断网站是否存活。

示例：![1570862031176](C:\Users\86155\Desktop\1570862031176.png)

## 3.curl -d "params" url

作用：使用-d发送带参数的请求（默认是post方式提交）

示例：![1570862245513](C:\Users\86155\Desktop\1570862245513.png)

## 4.curl -H Head_infos

作用：自定义Header头信息

示例：curl  -H  "User-Agent:Mozilla/5.0 (Windows NT 6.1; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/69.0.3497.100 Safari/537.36"  -H  "Referer:http://www.iqiyi.com"  http://vip.iqiyi.com

## 5.curl -L url

作用：跟踪链接url重定向，有些页面或接口被重定向，直接使用curl url会返回重定向的信息。使用curl -L url 可以跟踪。

## 6.curl -O url

作用：下载指定资源文件到当前目录中

示例：curl -O http://sr4.pplive.cn/cms/15/70/0dbb8ec002f1353e487d13fd949727a3.jpg  -O   http://sr4.pplive.cn/cms/39/88/4441be4257c21285c504509e479a43ea.jpg

## 7.curl -w [参数] url

**url_effective** 最终获取的url地址，尤其是当你指定给curl的地址存在301跳转，且通过-L继续追踪的情形。
**http_code** http状态码，如200成功,301转向,404未找到,500服务器错误等。(The numerical response code that was found in the last retrieved HTTP(S) or FTP(s) transfer. In 7.18.2 the alias response_code was added to show the same info.)
**http_connect** The numerical code that was found in the last response (from a proxy) to a curl CONNECT request. (Added in 7.12.4)
**time_total** 总时间，按秒计。精确到小数点后三位。 （The total time, in seconds, that the full operation lasted. The time will be displayed with millisecond resolution.）
time_namelookup DNS解析时间,从请求开始到DNS解析完毕所用时间。(The time, in seconds, it took from the start until the name resolving was completed.)
**time_connect** 连接时间,从开始到建立TCP连接完成所用时间,包括前边DNS解析时间，如果需要单纯的得到连接时间，用这个time_connect时间减去前边time_namelookup时间。以下同理，不再赘述。(The time, in seconds, it took from the start until the TCP connect to the remote host (or proxy) was completed.)
**time_appconnect** 连接建立完成时间，如SSL/SSH等建立连接或者完成三次握手时间。(The time, in seconds, it took from the start until the SSL/SSH/etc connect/handshake to the remote host was completed. (Added in 7.19.0))
**time_pretransfer** 从开始到准备传输的时间。(The time, in seconds, it took from the start until the file transfer was just about to begin. This includes all pre-transfer commands and negotiations that are specific to the particular protocol(s) involved.)
**time_redirect** 重定向时间，包括到最后一次传输前的几次重定向的DNS解析，连接，预传输，传输时间。(The time, in seconds, it took for all redirection steps include name lookup, connect, pretransfer and transfer before the final transaction was started. time_redirect shows the complete execution time for multiple redirections. (Added in 7.12.3))
**time_starttransfer** 开始传输时间。在发出请求之后，Web 服务器返回数据的第一个字节所用的时间(The time, in seconds, it took from the start until the first byte was just about to be transferred. This includes time_pretransfer and also the time the server needed to calculate the result.)
**size_download** 下载大小。(The total amount of bytes that were downloaded.)
**size_upload** 上传大小。(The total amount of bytes that were uploaded.)
**size_header** 下载的header的大小(The total amount of bytes of the downloaded headers.)
**size_request** 请求的大小。(The total amount of bytes that were sent in the HTTP request.)
**speed_download** 下载速度，单位-字节每秒。(The average download speed that curl measured for the complete download. Bytes per second.)
**speed_upload** 上传速度,单位-字节每秒。(The average upload speed that curl measured for the complete upload. Bytes per second.)
**content_type** 就是content-Type，不用多说了，这是一个访问我博客首页返回的结果示例(text/html; charset=UTF-8)；(The Content-Type of the requested document, if there was any.)
**num_connects** 最近的的一次传输中创建的连接数目。Number of new connects made in the recent transfer. (Added in 7.12.3)
**num_redirects** 在请求中跳转的次数。Number of redirects that were followed in the request. (Added in 7.12.3)
**redirect_url** When a HTTP request was made without -L to follow redirects, this variable will show the actual URL a redirect would take you to. (Added in 7.18.2)
**ftp_entry_path** 当连接到远程的ftp服务器时的初始路径。The initial path libcurl ended up in when logging on to the remote FTP server. (Added in 7.15.4)
**ssl_verify_result** ssl认证结果，返回0表示认证成功。( The result of the SSL peer certificate verification that was requested. 0 means the verification was successful. (Added in 7.19.0))





## 实例：



```shell
1.curl -I 127.0.0.1/xinnet.html -w "我爱你 ：%{http_code}" -s -o /dev/null
-I 获取请求头信息 不显示接口响应内容
-w %{http_code}  单独获取状态码
-s 静默模式
-o 把除-w之外的内容保存到黑洞（这样就达到了只取状态码的目的）
```







####不够完善  后期添加！

