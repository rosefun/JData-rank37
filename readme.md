京东JData算法大赛-高潜用户购买意向预测
===

**题目**：<http://www.datafountain.cn/#/competitions/247/data-intro><br><br>
**队名**：WOW<br><br>
**队员**：Lindada（队长）、hopehopehope、zhanggoudong<br><br>
**成绩排名**：<br>A榜：99/4241<br>	B榜：37/4241<br><br>

文件说明：
---
1. A_split_data.py，原始数据分割
2.  A_info_dict.py，数据预处理
3. A_fetch_feature.py，特征提取
4.  A_model.py，模型训练
5. A_valid_and_pred.py，验证与预测<br><br>

运行环境与工具：
---
ubuntu16.04<br>
python2.7、xgboost0.6<br>
numpy、pandas、sklearn<br><br>

思路简析：
===

数据清洗：
---
筛选出商品子集P中所有商品的历史记录，删除其他商品的历史记录。<br><br>

特征提取：
---
1.**用户行为特征**：<br>
预测日期之前10天内对商品子集有交互的所有用户，在距离预测日期第1天、第2天、第3天、前3天、前5天、前7天、前10天、前14天、前21天、前28天、前40天、前50天的浏览数、加入购物车次数、删除购物车次数、购买次数、关注次数、点击次数、总操作数、加入购物车和关注次数求和、购买和删除购物车次数求和、浏览次数与购买次数的比值、加入购物车次数与购买次数次数的比值、删除购物车与购买次数的比值、关注次数与购买次数的比值、点击次数与购买次数的比值；用户在前10天加入购物车的商品总数、关注的商品总数；用户在预测日期前50天出现操作行为的日期（有操作行为为1、没有为0）；用户最后一次浏览操作、加入购物车操作、删除购物车操作、购买操作、关注操作、点击操作距离预测日期的天数。<br><br>
2.**用户固有特征**:<br>
用户的年龄、性别、账号等级。<br><br>
3.**商品的行为特征**:<br>
预测日期之前10天内对用户有交互的所有商品，在距离预测日期第1天、第2天、第3天、前3天、前5天、前7天、前10天、前14天、前21天、前28天、前40天、前50天的被浏览数、加入购物车次数、删除购物车次数、购买次数、关注次数、点击次数、总操作数、加入购物车和关注次数求和、购买和删除购物车次数求和、浏览次数与购买次数的比值、加入购物车次数与购买次数次数的比值、删除购物车与购买次数的比值、关注次数与购买次数的比值、点击次数与购买次数的比值。<br><br>
4.**商品固有特征**:<br>
商品的属性1、属性2、属性3、品牌ID、评论数、有无差评、差评率<br><br>
5.**用户-商品对的关联行为特征**:<br>
预测日期之前10天内出现的所有用户商品组合，在距离预测日期第1天、第2天、第3天、前3天、前5天、前7天、前10天、前14天、前21天、前28天、前40天、前50天的被浏览数、加入购物车次数、删除购物车次数、购买次数、关注次数、点击次数、总操作数。<br><br>

数据采样与预处理：
---
1.选取3月27日-4月5日出现的所有用户/商品/用户-商品对在2月16日-4月5日50天中的数据提取上述特征，4月6日-10日用户-商品对是否发生购买行为作为标签。<br><br>
2.以天为时间粒度，时间窗口向前滑动。取10个时间窗口的数据Shuffle，构建数据集。<br><br>
3.构建出的数据集数据量庞大，为千万级的400维左右的特征样本，且正负样本比例严重失衡，大概为1:665。为了提高训练速度，又不降低训练的准确度。我们对负样本进行了5次又放回的下采样，以保证正负样本比例为1:14，分别训练5个分类器，并用5个分类器分别预测后投票得到最终结果。提高训练书读的同时尽量少的损失负样本信息。<br><br>

算法与模型：
---
最终选用xgboost模型。期间尝试过LR、RF、Adaboost、ETR、GBDT，不是分数和xgb相差较大就是训练速度较慢，因此最终选用xgboost算法。
模型选用的用户-商品对单模。由于我第一次参加比赛没有经验，队友工作比较忙，时间非常紧张，并没有使用多模型。赛后交流得知，排名靠前的队伍大都是用户模型和用户-商品模型融合，用户-商品单模达到一定分数就会比较难提高。<br><br>

验证与预测：
---
选取4月11日-4月15日的购买行为作为线下验证集。用上述所提数据采样方法得到5个数据子集，分别训练5个相同参数的xgboost模型，用得到的5个xgboost模型分别对验证集进行预测，得到的结果进行投票，得到最终的预测结果。线上预测与线下验证方法相同。比赛过程中，线上预测分数始终高于线下验证分数，A榜线上高0.01，B榜线上高0.04，但线下线上分数几乎同增同减。<br><br>

赛后小结：
===
第一次参加数据挖掘的比赛，收获很多，熟悉了拿到数据以后进行数据探索/数据清洗/特征提取/特征选择/模型构建/效果评估的整个过程，由于缺乏经验，期间走了很多弯路，代码也写得不够简洁。由于第一个月一直用笔记本在做，内存较小无法承受pandas的读写，只好用csv包按行读写、用numpy按行处理数据，代码冗长也耗费的大量的时间，不过最后结果还是不错的。最后，感谢 ***hope*** 姐的悉心指导，感谢 ***逆风的飞翔*** 几名师兄的热心帮助。
欢迎交流学习，如果此文对你的学习有帮助，欢迎star。

