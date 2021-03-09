# Rank feature query
根据rank_feature或rank_features字段的数值增加文档的相关性得分。

rank_feature查询通常用在bool查询的should子句中，因此它的相关分数被添加到bool查询的其他分数中。

与function_score查询或其他更改相关分数的方法不同，当track_total_hits参数不为真时，rank_feature查询可以有效地跳过非竞争的hits。这可以极大地提高查询速度。

## Rank feature functions
为了计算基于rank特征字段的相关分数，rank_feature查询支持以下数学函数:
* Saturation (饱和度)
* Logarithm （对数）
* Sigmoid