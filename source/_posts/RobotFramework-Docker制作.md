---
title: RobotFramework-Docker制作
copyright: true
date: 2018-04-27 14:29:52
tags:
- RobotFramework
- Wise2c
---

### Dockerfile

- 安装`Git`和运行RobotFramework测试用例需要的库
- `unic.py`文件主要添加了RobotFramework运行输出中文乱码的问题
- `send_robot_result_email.py`文件是解析RobotFramework运行结果中`output.xml`文件，然后把解析的结果整理成`html`格式通过邮件发送出来
- `run_robotframework.sh`文件是运行RobotFramework测试用例，然后运行`send_robot_result_email.py`脚本
- 配置定时执行脚本`send_robot_result_email.py`任务

```dockerfile
FROM python:2.7-alpine
LABEL maintainer="leicj <leicj@wise2c.com>"

#================================================
# 安装git，配置git的name和email
# 配置时区：Asia/Shanghai
#================================================
RUN apk --update add git tzdata && \
    git config --global user.email "leicj@wise2c.com" && \
    git config --global user.name "TestGroup" && \
    cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime && \
    apk del tzdata

#================================================
# 安装wise2c需要的robotframework库：selenium2library
#    requests,databaselibrary
# 其中pymysql是databaselibrary需要的插件
#================================================
RUN pip install -U \
    pip \
    pymysql \
    robotframework \
    robotframework-requests \
    robotframework-selenium2library \
    robotframework-databaselibrary

#================================================
# 拷贝unic.py，解决输出结果又中文乱码，添加utf-8编码
#================================================
COPY unic.py	/usr/local/lib/python2.7/site-packages/robot/utils/unic.py

#================================================
# 拷贝send_robot_result_email.py，主要用于解析运行结果并发送邮件
#================================================
COPY robotframework.py /send_robot_result_email.py

#================================================
# 拷贝run_robotframework.sh，运行wise2c UI自动化case，然后发出邮件
#================================================
COPY robot.sh   /run_robotframework.sh
RUN chmod +x /run_robotframework.sh

#================================================
# 配置定时执行运行脚本run_robotframework.sh
#================================================
ENV CRONJOB="0	3	*	*	*"
RUN echo "${CRONJOB}	sh /run_robotframework.sh" >> /etc/crontabs/root

ENV EMAIL_TO_LIST=leicj@wise2c.com,liaoxb@wise2c.com,weiht@wise2c.com,leji.chen@wise2c.com
VOLUME ["/automationtest"]
CMD ["crond -f"]

```

### Docker运行

- 运行以下`shell`命令

```shell
docker run -d --name chrome-browser -v /dev/shm:/dev/shm selenium/standalone-chrome
docker run -d --name wise2c-auto --link chrome-browser:web registry.cn-hangzhou.aliyuncs.com/xiaoyaoray/robotframework-wise2c # 可以加上 -v xxxx:/automationtest automationtest主要存放RobotFramework需要运行的Case
sh /run_robotframework.sh #进入到容器wise2c-auto中执行该脚本 
```

- `robotframework-wise2c`镜像中映射卷`/automationtest`存放RobotFramework需要运行的Case，执行脚本`/run_robotframework.sh`就是执行`/automationtest`目录下的测试用例
- 环境变量`CRONJOB`定时执行脚本`/run_robotframework.sh`表达式
- 环境变量`EMAIL_TO_LIST`收到邮件的人信息，不同邮箱间用`逗号`分开
- **PS:**默认情况下每天凌晨3点会执行脚本`/run_robotframework.sh`

### 附录

- `run_robotframework.sh`脚本

  ```shell
  git clone https://wise2c_leicj:qwer1234@gitee.com/wisecloud/automationtest.git /home/wise2c
  cp -r /home/wise2c/* /automationtest
  cd /home/wise2c
  git checkout log
  outputdir=`date +%Y%m%d-%H:%M:%S`
  web_ip=`env | grep "PORT_4444_TCP_ADDR" | awk -F = '{print $2}'`
  robot -v REMOTE_URL:http://$web_ip:4444/wd/hub -s ui -d /home/wise2c/$outputdir /automationtest
  cd /home/wise2c
  dirs=`ls -l /home/wise2c | grep "^d" | awk '{print $9}'| sort -r`
  i=0
  for name in $dirs
  do
      let i=i+1
      if [ i -ge 30];then
          rm -rf $name
      fi
  done
  git add .
  git commit -m $outputdir
  git push origin log
  python /send_robot_result_email.py /home/wise2c/$outputdir $EMAIL_TO_LIST
  rm -rf /home/wise2c
  rm -rf /automationtest/*
  ```

- `send_robot_result_email.py`脚本

  ```python
  #!/usr/bin/env python
  # -*- coding: utf-8 -*-

  import smtplib, datetime, sys
  from email.mime.text import MIMEText
  from email.header import Header
  from xml.etree import ElementTree


  def send_test_result_mail(receiver, robot_result, sender=None, password=None):
      """发送测试结果到邮箱
      """
      if not sender:
          sender = "leicj@wise2c.com"
          password = "N5SEnQz39AgHFibr"
      else:
          if not password:
              print "密码为空，请重试！"
              return

      mail_subject = "Automation Test Report"
      mail_smtpserver = "smtp.exmail.qq.com"
      mail_username = "leicj@wise2c.com"
      mail_password = "N5SEnQz39AgHFibr"
      mail_to_addrs = receiver.split(",")

      msg = MIMEText(robot_result, _subtype="html", _charset="utf-8")
      msg["Subject"] = Header(mail_subject, "utf-8")
      msg["From"] = "Wise2c-TestGroup"
      msg["To"] = msg["To"] = ",".join(mail_to_addrs)

      try:
          smtp = smtplib.SMTP_SSL()
          smtp.connect(mail_smtpserver, port=465)
          smtp.login(mail_username, mail_password)
          smtp.sendmail(mail_username, mail_to_addrs, msg.as_string())
      except:
          print "邮件发送失败！请检查配置，然后重试！"
      else:
          print "邮件发送成功！注意查收！"
      finally:
          smtp.quit()


  def _to_str(unicode_or_str):
      """将unicode字符串转换为str类型
      """
      if isinstance(unicode_or_str, unicode):
          value = unicode_or_str.encode('utf-8')
      else:
          value = unicode_or_str
      return value


  def _get_excdate(elment):
      start = elment.attrib.get("starttime")
      end = elment.attrib.get("endtime")
      start = datetime.datetime.strptime(start, "%Y%m%d %H:%M:%S.%f")
      end = datetime.datetime.strptime(end, "%Y%m%d %H:%M:%S.%f")
      m, s = divmod((end - start).seconds, 60)
      if int(m) == 0:
          excdate = "%s秒" % s
      elif int(s) == 0:
          excdate = "%s分" % m
      else:
          excdate = "%s分%s秒" % (m, s)
      return excdate


  def _parse_results_xml(xml_path):
      """RobotFramework运行自动化测试用例结果解析
      """
      tree = ElementTree.parse(xml_path)
      results = {}
      fail_cases = []

      root = tree.getroot().attrib["generated"]
      run_date = datetime.datetime.strptime(root, "%Y%m%d %H:%M:%S.%f")
      run_date = datetime.datetime.strftime(run_date, "%Y-%m-%d %H:%M:%S")
      results["run_date"] = run_date
      for n in tree.findall("suite"):
          for y in n.findall("status"):
              results["suite_date"] = _get_excdate(y)
              if y.attrib["status"] == "pass":
                  results["status"] = True
              else:
                  results["status"] = False

      for i in tree.getiterator("total"):
          results["fail_num"] = int(i.find("stat").attrib.get("fail"))
          results["pass_num"] = int(i.find("stat").attrib.get("pass"))
          results["total_num"] = results["fail_num"] + results["pass_num"]

      if not results["status"]:
          for test in tree.getiterator("test"):
              if test.find("status").attrib.get("status") == "FAIL":
                  test_info = test.attrib
                  test_info["excDate"] = _get_excdate(test.find("status"))
                  fail_cases.append(test_info)
          results["fail_cases"] = fail_cases
      return results


  output_xml = sys.argv[1]
  url = "http://wisecloud.gitee.io/automationtest/" + output_xml[13:]
  res = _parse_results_xml(output_xml + "/output.xml")

  base_styles = """
  /*base css*/
  a{color:#4a72af}
  body{background-color:#e4e4e4}
  body,p{margin:0;padding:0}
  img{display:block}
  h1,h2,h3,h4,h5,h6{margin:0 0 .8em 0}
  h3{font-size:28px;color:#444!important;font-family:Arial,Helvetica,sans-serif}
  h4{font-size:22px;color:#4a72af!important;font-family:Arial,Helvetica,sans-serif}
  h5{font-size:18px;color:#444!important;font-family:Arial,Helvetica,sans-serif}
  p{font-size:12px;color:#444!important;font-family:"Lucida Grande","Lucida Sans","Lucida Sans Unicode",sans-serif;line-height:1.5}

  table.robotstat {
    border: 1px solid black;
    border-collapse: collapse;
    empty-cells: show;
    margin: 0px 1px;
    table-layout: fixed;
    word-wrap: break-word;
    font-size: 1em;
    width:100%;
  }

  table.robotstat td{
      border: 1px solid #ddd;
      padding: 4px;
      line-height: 1.42857143;
  }

  tr.test_column_robot {
    background-color：#C6C6C6;
    border: 1px solid black;
  }

  ol li img{display:inline;height:20px}"""

  div_styles = """
  /*div styles*/
  .news{text-align:center;padding-top:15px;}
  .content{width:720px;margin:0 auto;background-color:white}
  .round_border{margin-bottom:5px;-webkit-border-radius:6px;-moz-border-radius:6px;border-radius:6px;margin-top:0;font-size:14px;padding:6px;border:1px solid #ccc}
  .status .info{color:white!important;text-shadow:0 -1px 0 rgba(0,0,0,0.3);font-size:32px;line-height:36px;padding:8px 0}
  .main img{width:38px;margin-right:16px;height:38px}
  .main table{font-size:14px;}
  .main table th{text-align:right;}
  .bottom-message{width:720px;cellpadding:5px;cellspacing:0px}
  .bottom-message .message{font-size:13px;color:#aaa;line-height:18px;text-align:center}
  .bottom-message .designed{font-size:13px;color:#aaa;line-height:18px;font-style: italic;text-align:right}
  img.cartoon {width: 36px; display:inline}
  """
  if res["status"]:
      build_status_color = "green"
  else:
      build_status_color = "red"
  div_styles += ".status{background-color:%s;font-size:28px;font-weight:bold;color:white;width:720px;height:52px;margin-bottom:18px;text-align:center;vertical-align:middle;border-collapse:collapse;background-repeat:no-repeat}" % build_status_color

  msg_styles = "<STYLE>" + base_styles + div_styles + "</STYLE>"


  msg_body = """<DIV class="content round_border">
  <DIV class="status">"""
  if res["status"]:
      test_res = "成功"
  else:
      test_res = "失败"
  msg_body += """<h4 class="info">测试结果:%s</h4>""" % test_res

  msg_body += """
  </DIV>
  <TABLE>
    <TR><TD>Project:</TD><TD>WiseCloud自动化测试</TD></TR>"""
  msg_body += """
    <TR><TD>Date of run:</TD><TD>%s</TD></TR>""" % res["run_date"]
  msg_body += """
    <TR><TD>Test duration:</TD><TD>%s</TD></TR> """ % res["suite_date"]
  msg_body += """
    <TR><TD>Test report:</TD><TD><A href="%s/report.html">Open report.html</A></TD></TR>""" % url
  msg_body += """
    <TR><TD>Test log:</TD><TD><A href="%s/log.html">Open log.html</A></TD></TR>""" % url
  msg_body += """
  </TABLE>
  </DIV>
  <BR/>
  """
  msg_body += """
  <!-- Robot Framework Results -->
  <DIV class="content round_border">
  <p><h4>Robot Framework Results</h4></p>
  <table class="robotstat">
          <thead>
            <tr class="test_column_robot">
              <td><b>Total</b></td>
              <td><b>Passed</b></td>
              <td><b>Failed</b></td>
              <td><b>Pass %</b></td>
            </tr>
          </thead>
          <tbody>
  """

  msg_body += "<td>%s</td>" % res["total_num"]
  msg_body += "<td>%s</td>" % res["pass_num"]
  msg_body += "<td>%s</td>" % res["fail_num"]
  percent = float(res["pass_num"]) * 100 / float(res["total_num"])
  msg_body += "<td>%0.2f</td>" % percent

  msg_body += """
            </tr>
          </tbody>
        </table>
  </DIV>
  <BR/>
  """
  if res["status"]:
      msg_body += """
      <DIV class="content round_border">
      <h4>No Robot Framework test results found.</h4>
      </DIV>
      """

  if not res["status"]:
      msg_body += """
      <DIV class="content round_border">
      <p><h4>Robot Framework Error Cases</h4></p>
      <table class="robotstat">
      <thead>
      <tr class="test_column_robot">
      <td><b>Test Name</b></td>
      <td><b>Status</b></td>
      <td><b>Execution Datetime</b></td>
      </tr>
      </thead>
      """
      msg_body += """
      <tbody>"""
      for case in res["fail_cases"]:
          case_name = _to_str(case["name"])
          # msg_body += """
          # <tr><td colspan="3"><b>%s</b></td></tr>""" % case_name
          msg_body += """
          <tr>"""
          msg_body += """
          <td><a href="%s/log.html#%s">%s</a></td>""" % (url, case["id"], case_name)
          msg_body += """
          <td style="color: #FF3333">Fail</td>
          """
          msg_body += """
          <td>%s</td>""" % case["excDate"]
          msg_body += """
          </tr>"""
      msg_body += """
      </tbody></table>
      """

  msg_body = "<BODY>" + msg_body + "</BODY>"
  msg_head = """<head>
          <meta charset="utf-8">
          <title></title>
      </head>"""

  msg = """<!DOCTYPE html>
  <html>"""
  msg = msg + msg_head + msg_styles + msg_body + "</html>"

  send_test_result_mail(sys.argv[2], msg)
  # with open("~/Desktop/robot_result.html", "w") as f:
  #     f.write(msg)


  #send_robotframework_mail("leicj@wise2c.com", msg)
  ```

- `unic.py`脚本

  ```python
  #  Copyright 2008-2015 Nokia Networks
  #  Copyright 2016-     Robot Framework Foundation
  #
  #  Licensed under the Apache License, Version 2.0 (the "License");
  #  you may not use this file except in compliance with the License.
  #  You may obtain a copy of the License at
  #
  #      http://www.apache.org/licenses/LICENSE-2.0
  #
  #  Unless required by applicable law or agreed to in writing, software
  #  distributed under the License is distributed on an "AS IS" BASIS,
  #  WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
  #  See the License for the specific language governing permissions and
  #  limitations under the License.

  from pprint import PrettyPrinter

  from .platform import IRONPYTHON, JYTHON, PY2
  from .robottypes import is_bytes, is_unicode
  import json


  if PY2:

      def unic(item):
          if isinstance(item, unicode):
              return item
          if isinstance(item, (bytes, bytearray)):
              try:
                  #return item.decode('ASCII')
                  return item.decode('utf-8')
              except UnicodeError:
                  return u''.join(chr(b) if b < 128 else '\\x%x' % b
                                  for b in bytearray(item))
  	if isinstance(item, (list, dict, tuple)):
  	     try:
  		item = json.dumps(item, ensure_ascii=False, encoding='cp936')
               except UnicodeDecodeError:
  	         try:
  		     item = json.dumps(item, ensure_ascii=False, encoding='cp936')
  	         except:
  		     	pass
  	     except:
  		 	pass
          # 添加部分
          try:
              try:
                  return unicode(item)
              except UnicodeError:
                  return unic(str(item))
          except:
              return _unrepresentable_object(item)

  else:

      def unic(item):
          if isinstance(item, str):
              return item
          if isinstance(item, (bytes, bytearray)):
              try:
                  #return item.decode('ASCII')
                  return item.decode('utf-8')
              except UnicodeError:
                  return ''.join(chr(b) if b < 128 else '\\x%x' % b
                                 for b in item)
          try:
              return str(item)
          except:
              return _unrepresentable_object(item)


  # JVM and .NET seem to handle Unicode normalization automatically. Importing
  # unicodedata on Jython also takes some time so it's better to avoid it.
  if not (JYTHON or IRONPYTHON):

      from unicodedata import normalize
      _unic = unic

      def unic(item):
          return normalize('NFC', _unic(item))


  def prepr(item, width=400):
      return unic(PrettyRepr(width=width).pformat(item))


  class PrettyRepr(PrettyPrinter):

      def format(self, object, context, maxlevels, level):
          try:
              if is_unicode(object):
                  return repr(object).lstrip('u'), True, False
              if is_bytes(object):
                  return 'b' + repr(object).lstrip('b'), True, False
              return PrettyPrinter.format(self, object, context, maxlevels, level)
          except:
              return _unrepresentable_object(object), True, False


  def _unrepresentable_object(item):
      from .error import get_error_message
      return u"<Unrepresentable object %s. Error: %s>" \
             % (item.__class__.__name__, get_error_message())

  ```

  ​