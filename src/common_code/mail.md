# 邮件

## 发送邮件

函数：

```python

def sendEmail(  sender, senderPassword, receiverList,
                senderName="", receiverNameList= "",
                smtpServer = "", smtpPort = None, useSSL=False,
                type = "plain", title = "", body = ""):
    """
    send email
    :param sender:
    :param senderPassword:
    :param receiverList:
    :param senderName:
    :param receiverNameList:
    :param smtpServer:
    :param smtpPort:
    :param type: html/plain
    :param title:
    :param body:
    :return:
    """
    logging.debug("sender=%s, senderName=%s, smtpServer=%s, smtpPort=%s, useSSL=%s, type=%s, title=%s, body=%s",
                  sender, senderName, smtpServer, smtpPort, useSSL, type, title, body)
    logging.debug("receiverList=%s, receiverNameList=%s", receiverList, receiverNameList)

    defaultPort = None
    SMTP_PORT_NO_SSL = 25
    SMTP_PORT_SSL = 465
    if useSSL:
        defaultPort = SMTP_PORT_SSL
    else:
        defaultPort = SMTP_PORT_NO_SSL

    if not smtpPort:
        smtpPort = defaultPort

    # init smtp server if necessary
    if not smtpServer:
        # extract domain from sender email
        # crifan2003@163.com -> 163.com
        atIdx = sender.index('@')
        afterAtIdx = atIdx + 1
        lastDomain = sender[afterAtIdx:]
        smtpServer = 'smtp.' + lastDomain
        # smtpServer = "smtp.163.com"
        # smtpPort = 25

    # RECEIVER_SEPERATOR = '; '
    RECEIVER_SEPERATOR = ', '

    senderNameAddr = "%s <%s>" % (senderName, sender)
    receiversAddr = RECEIVER_SEPERATOR.join(receiverList)
    receiverNameAddrList = []
    formatedReceiverNameAddrList = []
    for curIdx, eachReceiver in enumerate(receiverList):
        eachReceiverName = receiverNameList[curIdx]
        eachNameAddr = "%s <%s>" % (eachReceiverName, eachReceiver)
        eachFormatedNameAddr = formatEmailNameAddrHeader(eachNameAddr)
        receiverNameAddrList.append(eachNameAddr)
        formatedReceiverNameAddrList.append(eachFormatedNameAddr)

    formatedReceiversNameAddr = RECEIVER_SEPERATOR.join(formatedReceiverNameAddrList) # '=?utf-8?b?57u/6Imy5Z6D5Zy+?= <green-waste@163.com>, =?utf-8?b?5YWL55Ge6Iqs?= <admin@crifan.com>'
    mergedReceiversNameAddr = RECEIVER_SEPERATOR.join(receiverNameAddrList) # u'Crifan2003 <crifan2003@163.com>, 克瑞芬 <admin@crifan.com>'
    # formatedReceiversNameAddr = formatEmailHeader(mergedReceiversNameAddr) #=?utf-8?b?Q3JpZmFuMjAwMyA8Y3JpZmFuMjAwM0AxNjMuY29tPiwg5YWL55Ge6IqsIDxh?=
    # =?utf-8?q?dmin=40crifan=2Ecom=3E?=

    msg = MIMEText(body, _subtype=type, _charset="utf-8")
    # msg["From"] = _format_addr(senderNameAddr)
    # msg["To"] = _format_addr(receiversNameAddr)
    msg["From"] = formatEmailHeader(senderNameAddr)
    # msg["From"] = senderNameAddr
    # msg["To"] = formatEmailHeader(formatedReceiversNameAddr)
    # msg["To"] = formatedReceiversNameAddr
    # msg["To"] = mergedReceiversNameAddr
    # msg["To"] = formatEmailHeader(receiversAddr)
    msg["To"] = formatEmailHeader(mergedReceiversNameAddr)
    # titleHeader = Header(title, "utf-8")
    # encodedTitleHeader = titleHeader.encode()
    # msg['Subject'] = encodedTitleHeader
    msg['Subject'] = formatEmailHeader(title)
    # msg['Subject'] = title
    msgStr = msg.as_string()

    # try:
    # smtpObj = smtplib.SMTP('localhost')
    smtpObj = None
    if useSSL:
        smtpObj = smtplib.SMTP_SSL(smtpServer, smtpPort)
    else:
        smtpObj = smtplib.SMTP(smtpServer, smtpPort)
        # start TLS for security
        # smtpObj.starttls()
    # smtpObj.set_debuglevel(1)
    smtpObj.login(sender, senderPassword)
    # smtpObj.sendmail(sender, receiversAddr, msgStr)
    smtpObj.sendmail(sender, receiverList, msgStr)
    logging.info("Successfully sent email: message=%s", msgStr)
    # except smtplib.SMTPException:
    #     logging.error("Fail to sent email: message=%s", message)

    return
```

调用：

```python
productName = "First 163 then crifan. Dell XPS 13 XPS9360-5797SLV-PUS Laptop"
productUrl = "https://www.microsoft.com/en-us/store/d/dell-xps-13-xps-9360-laptop-pc/8q17384grz37/GV5D?activetab=pivot%253aoverviewtab"
notifType = "HighPrice"
title = "[%s] %s" % (notifType, productName)
notifContent = """
<html>
    <body>
        <h1>%s</h1>
        <p>Not buy <a href="%s">%s</a> for current price <b>$699.00</b> &gt; expected price <b>$599.00</b></p>
        <p>So save for later process</p>
    </body>
</html>
""" % (title, productUrl, productName)
receiversDictList = gCfg["notification"]["receivers"]
receiverList = []
receiverNameList = []
for eachReceiverDict in receiversDictList:
    receiverList.append(eachReceiverDict["email"])
    receiverNameList.append(eachReceiverDict["username"])

sendEmail( 
    sender = gCfg["notification"]["sender"]["email"],
    senderPassword = gCfg["notification"]["sender"]["password"],
    receiverList = receiverList,
    senderName = gCfg["notification"]["sender"]["username"],
    receiverNameList= receiverNameList,
    type = "html",
    title = title,
    body = notifContent
)
```

附录：

* 最新代码详见：
  * https://github.com/crifan/crifanLibPython/blob/master/python3/crifanLib/crifanEmail.py

