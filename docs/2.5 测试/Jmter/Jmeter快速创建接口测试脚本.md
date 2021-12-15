　**操作流程**

[![img](http://www.51testing.com/attachments/2020/08/15326825_2020082713325312NgF.png)](http://www.51testing.com/batch.download.php?aid=115886)

　　启动Jmeter。

　　**1.添加“线程组”**

　　右击“测试计划”，选择“添加”\“线程（用户）”\“线程组”

[![img](http://www.51testing.com/attachments/2020/08/15326825_202008271333221NeDY.png)](http://www.51testing.com/batch.download.php?aid=115893)

　　**2.添加/设置“HTTP请求”**

[![img](http://www.51testing.com/attachments/2020/08/15326825_202008271333561xxHk.png)](http://www.51testing.com/batch.download.php?aid=115899)

　　右击“线程组”，选择“添加”\“取样器”\”HTTP请求”。

　　根据接口文档要求，填写相应参数，例如https://www.juhe.cn/docs/api/id/39（全国天气查询），填到“HTTP请求”中。

[![img](http://www.51testing.com/attachments/2020/08/15326825_202008271337501SDjs.png)](http://www.51testing.com/batch.download.php?aid=115902)

[![img](http://www.51testing.com/attachments/2020/08/15326825_202008271338051CLOL.png)](http://www.51testing.com/batch.download.php?aid=115903)

　　填写说明：

　　服务器名称或IP：输入域名(例如：www.juhe.cn），此处不需要设置端口，如果有端口还需要填写端口。

　　方法:选择“GET”

　　路径：输入接口地址中域名后路径部分（例如：/docs/api/id/39）

　　参数：输入cityname、dtype、format、key（来源聚合数据平台中该订单的AppKey）

[![img](http://www.51testing.com/attachments/2020/08/15326825_202008271339031aRrj.png)](http://www.51testing.com/batch.download.php?aid=115904)

　　3.**添加“察看结果树”**

　　右击“线程组”，选择“添加”\“监听器”\”察看结果树”。

[![img](http://www.51testing.com/attachments/2020/08/15326825_2020082713393912RH3.png)](http://www.51testing.com/batch.download.php?aid=115905)

　　保存脚本，点击三角形执行图标。

[![img](http://www.51testing.com/attachments/2020/08/15326825_202008271339551gLjY.png)](http://www.51testing.com/batch.download.php?aid=115906)

　　点击“察看结果树”，查看执行结果。

[![img](http://www.51testing.com/attachments/2020/08/15326825_202008271340011Ey5G.png)](http://www.51testing.com/batch.download.php?aid=115907)