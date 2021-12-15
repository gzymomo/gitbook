- [python数据统计之禅道bug统计](https://www.cnblogs.com/judede/p/15152845.html)



# 背景

通过定期输出 每条产品的 BUG 情况，以此来反馈开发解决问题、测试跟进问题的情况；钉钉群推送提醒开发及时解决

以此我这边开始着手准备编写一个小工具，最终达到目的：自动定期发送统计报告，报告维度（数据 + html展示）。

# 技术选型

python + markdown + pymysql + html + jenkins + 钉钉机器人

# 实现思路

python主要用到sshtunnel，pysql库跳板机连接mysql数据库，loguru库日志记录，yaml记录项目参数，request调取钉钉接口

1. 读取禅道数据库数据及状态，封装sql类，把各维度统计数据通过dic格式返回
2. 禅道bug汇总数据进行进行拼接，生成模板写入markdown文件。（钉钉会支持简单markdown格式，表格等等不支持）
3. 禅道bug明细数据，生成html页面（没找到合适的三方库只能手动撸）
4. 调取钉钉自定义接口，进行数据请求。
5. jenkins实现定期推送+html页面展示

# 主要代码

1、禅道sql封装

ps.公司服务国外的所以时区+8H

```sql
from datetime import datetime
from middleware.config_handler import db_connect
"""针对产品返回所有bug信息"""
def sql_pakeage(productid):
    """
    BUG状态统计SQL封装
    :param productid:
    :return:
    """
    bug_sql = "select count(*) from zt_bug where product in %s and deleted='0'"% (
    productid)

    resolved_bug_sql = "select count(*) from zt_bug where product in %s and deleted = '0' and `status` = 'resolved' and resolution <> 'postponed' " % (
        productid)

    not_resolved_bug_sql = "select count(*) from zt_bug where product in %s  and deleted = '0' and `status` =  'active' " % (
        productid)

    postponed_bug_sql = "select count(*) from zt_bug where product in %s and deleted = '0' and `status` <> 'closed' and resolution = 'postponed' " % (
        productid)

    closed_bug_sql = "select count(*) from zt_bug where product in %s and deleted = '0' and `status` = 'closed' " % (
        productid)

    return  bug_sql,resolved_bug_sql,not_resolved_bug_sql,postponed_bug_sql,closed_bug_sql


def  test_product_bug(productid):
    """
    产品BUG情况统计
    :param productid:
    :return:
    """
    #总bug数
    all_bug=db_connect.query_sql_df(sql_pakeage(productid)[0])

    #已解决bug数
    resolved_bug = db_connect.query_sql_df(sql_pakeage(productid)[1])

    # 未解决BUG数（当前显示BUG状态为未解决的。包含当前还没被解决的、之前遗留的未解决、以及reopen的BUG（累计数据））
    not_resolved_bug = db_connect.query_sql_df(sql_pakeage(productid)[2])

    # 延期BUG数
    postponed_bug= db_connect.query_sql_df( sql_pakeage(productid)[3])

    # 已关闭BUG数
    closed_bug = db_connect.query_sql_df(sql_pakeage(productid)[4])

    statistics_bug = { "总BUG数":all_bug[0],"已解决BUG": resolved_bug[0], "未解决BUG": not_resolved_bug[0], "已关闭BUG": closed_bug[0],
                      "延期解决BUG": postponed_bug[0]}
    print(statistics_bug)

    return  statistics_bug

def test_product_bug_near(day, product):
    """
        最近总的BUG情况统计统计
        :param: day 根据输入天数
        :return:
        """
    now = (datetime.datetime.now() + datetime.timedelta(hours=8)).strftime('%Y-%m-%d %H:%M:%S') #服务器时区要 + 8h
    recent_sevenday = (datetime.datetime.now() - datetime.timedelta(days=day)).strftime("%Y-%m-%d %H:%M:%S")
    new_near_bug = db_connect.query_sql_df("""SELECT count(*) from zt_bug b where  b.product in %s and openedDate between "%s" and "%s";"""%(product, recent_sevenday, now))
    open_bug = db_connect.query_sql_df("""SELECT count(*) from zt_bug b where  b.product  in %s and  b.STATUS = "active" and openedDate between "%s" and "%s";"""%(product, recent_sevenday, now))
    close_bug = db_connect.query_sql_df("""SELECT count(*) from zt_bug b where  b.product  in %s and  b.STATUS = "closed" and openedDate between "%s" and "%s";"""%(product, recent_sevenday, now))
    close_unbug = db_connect.query_sql_df("""SELECT count(*) from zt_bug b where  b.product  in %s and  b.STATUS = "resolved" and openedDate between "%s" and "%s";"""%(product, recent_sevenday, now))
    statistics_bug = { "本周新增BUG数":new_near_bug[0], "本周未解决BUG数":open_bug[0],"本周已解决BUG数":close_bug[0],"本周已解决待验证BUG数":close_unbug[0]}
    return statistics_bug

def bug_count(day, product):
    """
    最近总的BUG情况统计明细数据
    :param: day 根据输入天数
    :return:
    """
    now = (datetime.datetime.now() + datetime.timedelta(hours=8)).strftime('%Y-%m-%d %H:%M:%S') #服务器时区要 + 8h
    now_day = datetime.datetime.now().strftime('%Y-%m-%d')
    recent_sevenday = (datetime.datetime.now()-datetime.timedelta(days=day)).strftime("%Y-%m-%d %H:%M:%S")
    bug_sql = """
SELECT p.name,b.title,b.assignedTo,b.severity,b.type,b.status,b.openedBy, CAST(openedDate AS CHAR) AS openedDate from zt_bug b left join zt_product p on b.product = p.id 
where b.STATUS <> 'closed' and b.product in %s and openedDate between "%s" and "%s";"""%(product, recent_sevenday, now)
    print(bug_sql)
    recent_sevenday_bug = db_connect.query_sql_df(bug_sql)
    return recent_sevenday_bug
```

2、钉钉接口推送

```python
import json
import urllib.request

from config.path_config import data_file
from handler.loguru_handler import logger
def send_bug(url, data_file):
    # url = r"https://oapi.dingtalk.com/robot/send?access_token=XXXXXXX"
    logger.info("构建钉钉机器人的webhook")
    header = {"Content-Type": "application/json","Charset": "UTF-8"}

    with open(data_file, "r", encoding="utf-8") as f:
        conntent = f.read()

    data = {
        "msgtype": "markdown",
        "markdown": {
            "title": "测试bug统计",
            "text": conntent
        },
        "at": {
             "isAtAll": False     #@全体成员（在此可设置@特定某人）
        }
    }
    logger.info(f"构建请求内容:{data}")
    sendData = json.dumps(data)
    sendData = sendData.encode("utf-8")
    request = urllib.request.Request(url=url, data=sendData, headers=header)
    logger.info("发送请求")
    opener = urllib.request.urlopen(request)
    logger.info("请求求发回的数据构建成为文件格式")
    res = opener.read()
    logger.info(f"返回结果为:{res}")
```

3、bug明细转html文件

```python
def bug_html(lis ,html_file):
    """
    对查询bug明细转html文件
    :param lis
    :param html_file
    """
    conten_title = []
    for key in lis[0]:
        conten_title.append(key)
    a = "</th><th>".join(conten_title)
    con_title = "<tr><th>" + a + "</th></tr>"
    conten_val = []
    con = ""
    for i in range(0, len(lis)):
        for index, v in enumerate(lis[i]):
            if index ==0:
                lis[i][v] ="<tr><td>" + lis[i][v]
            con = con + str(lis[i][v]) + "</td><td>"
        con = con[0:-2] +"r>"
        con = con + "\n"
    head = """<meta charset="utf-8">
    <style type="text/css">
    table.tftable {font-size:12px;color:#333333;width:100%;border-width: 1px;border-color: #9dcc7a;border-collapse: collapse;}
    table.tftable th {font-size:12px;background-color:#abd28e;border-width: 1px;padding: 8px;border-style: solid;border-color: #9dcc7a;text-align:left;}
    table.tftable tr {background-color:#ffffff;}
    table.tftable td {font-size:12px;border-width: 1px;padding: 8px;border-style: solid;border-color: #9dcc7a;}
    </style>\n<table id="tfhover" class="tftable" border="1">\n"""
    last =  "</table>"
    htm = head + con_title + con + last

    with open(html_file, "w", encoding="utf-8") as f:
        f.write(htm)
```

另外功能基类以及调用发送代码比较简单就不展示了。

# 业务统计

## 按周统计(月份统计同理):

新增，新增日期为本周内的（包括本周内被解决或关闭的 BUG）

已解决，解决日期为本周内的。被开发设定为已解决的。其中可能有部分是上周遗留下来的，体现了开发在本周的变化情况（包括设计如此、重复 BUG、外部原因、无法重现、不予解决、转为需求），不包含延期处理

已关闭，关闭日期为本周内的。是测试验证过，确实已经解决的，包括其中有的是上周遗留下来的

未解决，当前显示 BUG 状态为未解决的。包含当前还没被解决的、之前遗留的未解决、以及 reopen 的 BUG（累计数据）

延期解决，当前显示 BUG 状态为 延期处理的。BUG 状态中新增一个延期解决 (累计数据)

# 应用截图

钉钉通知

[![img](https://img2020.cnblogs.com/blog/2078093/202108/2078093-20210817161840426-927514422.png)](https://img2020.cnblogs.com/blog/2078093/202108/2078093-20210817161840426-927514422.png)

明细数据展示

[![img](https://img2020.cnblogs.com/blog/2078093/202108/2078093-20210817162013928-2050352434.png)](https://img2020.cnblogs.com/blog/2078093/202108/2078093-20210817162013928-2050352434.png)



 