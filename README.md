EntitySimulator
========================

一个模拟Entity对Entity数据库表进行操作的工具。

部分代码和设计理论来自django，任何与django相关的内容请参考以下内容：
https://docs.djangoproject.com/en/1.11/ref/models/querysets/#queryset-api
https://docs.djangoproject.com/en/1.11/topics/db/queries/#complex-lookups-with-q
https://docs.djangoproject.com/en/1.11/ref/models/expressions/#django.db.models.F



安装方法：
---------------------
	1.更新common目录下所有内容并复制到KBEngine引擎的“kbe\res\scripts\common\”下面（当然，复制到自身脚本层上面也可以）。
	2.由于MysqlUtility.py用到了Mysql-Connector的一些方法，因此需要到https://cdn.mysql.com//Downloads/Connector-Python/mysql-connector-python-2.1.6.zip处下载python版本的mysql-connector。
	3.解压缩mysql-connector-python-2.1.6.zip并复制“mysql-connector-python-2.1.6\lib\mysql”目录到KBEngine引擎的“kbe\res\scripts\common\”下面。
	4.over.



测试代码：
---------------------
	from EntitySimulator.EntityModel import EntityModel
	from EntitySimulator import Fields

	class TestTable( EntityModel ):
		class Meta:
			db_table = "custom_TestTable"

		databaseID = Fields.INT32( db_column = "id", primary_key = True )
		i1         = Fields.INT32( db_column = "sm_i1" )
		i2         = Fields.FLOAT( db_column = "sm_i2" )
		sm_s1      = Fields.UNICODE(default = "test1")
		sm_s2      = Fields.BLOB(default = b'gdsadf')



telnet localhost 40000
---------------------
	from EntitySimulator.utils.query_utils import Q
	from EntitySimulator.utils.expressions import F

	g_result = []
	def cb(success, models): g_result.append((success, models))

	TestTable.objects.select(cb)

	# 等于
	TestTable.objects.select(cb, i1 = 1)

	# 大于
	TestTable.objects.select(cb, i1__gt = 1)

	# 大于等于
	TestTable.objects.select(cb, i1__gte = 1)

	# 小于
	TestTable.objects.select(cb, i1__lt = 1)

	# 小于等于
	TestTable.objects.select(cb, i1__lte = 1)

	# 值是1、3或7
	TestTable.objects.select(cb, i1__in = (1,3,7))

	# 在范围 1 - 10 之间
	TestTable.objects.select(cb, i1__range = (1, 10))

	# xxx ILIKE 'yyy'
	TestTable.objects.select(cb, sm_s1__iexact = "abc")
	TestTable.objects.select(cb, sm_s1__iexact = None)

	# 小于等于10或大于等于100
	TestTable.objects.select(cb, Q(i1__lte = 10) | Q(i2__gte = 100))

	# order by 以及 limit
	TestTable.objects.order_by("databaseID", "-i1").limit(1, 10).select(cb, databaseID = 100)

	TestTable.objects.filter(databaseID = 1).update(cb, i1 = 1111, sm_s1 = "cba")
	TestTable.objects.filter(databaseID = 123).update(cb, i1 = 220 + F("i1") + 110, sm_s1 = "cba")
	TestTable.objects.filter(databaseID = 456).update(cb, i1 = 220 + F("i1") + F("i2") + 110, sm_s1 = "cba")


	# 写入数据
	m = TestTable(i1 = 12, i2 = 34.0, sm_s1 = "abc", sm_s2 = b"gggtttvvv")
	m.writeToDB(cb)
	m.databaseID

	TestTable.objects.filter(databaseID = m.id).update(cb, i1 = 1111, sm_s1 = "cba")
	TestTable.objects.filter(databaseID = m.id).update(cb, i1 = F("i1") + 110, sm_s1 = "cba")

	# update custom_TestTable set sm_i1 = 220 + sm_i1 + sm_i2 + 110, sm_s1 = "cha" where id = xxx
	TestTable.objects.filter(databaseID = m.id).update(cb, i1 = 220 + F("i1") + F("i2") + 110, sm_s1 = "cba")
