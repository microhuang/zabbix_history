zabbix_history
==============

历史同期对比曲线图


一、现有结构

graphs_items

#图表配置数据

```
| graphs_items | CREATE TABLE `graphs_items` (
  `gitemid` bigint(20) unsigned NOT NULL,
  `graphid` bigint(20) unsigned NOT NULL,
  `itemid` bigint(20) unsigned NOT NULL,
  `drawtype` int(11) NOT NULL DEFAULT '0',
  `sortorder` int(11) NOT NULL DEFAULT '0',
  `color` varchar(6) NOT NULL DEFAULT '009600',
  `yaxisside` int(11) NOT NULL DEFAULT '0',
  `calc_fnc` int(11) NOT NULL DEFAULT '2',
  `type` int(11) NOT NULL DEFAULT '0',
  PRIMARY KEY (`gitemid`),
  KEY `graphs_items_1` (`itemid`),
  KEY `graphs_items_2` (`graphid`),
  CONSTRAINT `c_graphs_items_2` FOREIGN KEY (`itemid`) REFERENCES `items` (`itemid`) ON DELETE CASCADE,
  CONSTRAINT `c_graphs_items_1` FOREIGN KEY (`graphid`) REFERENCES `graphs` (`graphid`) ON DELETE CASCADE
) ENGINE=InnoDB DEFAULT CHARSET=utf8 |
```

CLineGraphDraw->addItem()

#获取graphs的item配置数据

CLineGraphDraw->selectData()

#根据itemid获取数据


二、改造

1、添加同期配置表

```
| graphs_items_history | CREATE TABLE `graphs_items_history` (
  `gitemid` bigint(20) unsigned NOT NULL,
  `graphid` bigint(20) unsigned NOT NULL,
  `itemid` bigint(20) unsigned NOT NULL,
  `drawtype` int(11) NOT NULL DEFAULT '0',
  `sortorder` int(11) NOT NULL DEFAULT '0',
  `color` varchar(6) NOT NULL DEFAULT '009600',
  `yaxisside` int(11) NOT NULL DEFAULT '0',
  `calc_fnc` int(11) NOT NULL DEFAULT '2',
  `type` int(11) NOT NULL DEFAULT '0',
  `is_history` int(11) NOT NULL DEFAULT '1',
  PRIMARY KEY (`gitemid`),
  KEY `graphs_items_1` (`itemid`),
  KEY `graphs_items_2` (`graphid`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 |
```

2、读取history配置

$dbGraphItems = DBselect(
        'SELECT gi.*'.
        ' FROM graphs_items_history gi'.
        ' WHERE gi.graphid='.zbx_dbstr($dbGraph['graphid']).
        ' ORDER BY gi.sortorder,gi.itemid DESC'
);
while ($dbGraphItem = DBfetch($dbGraphItems)) {
        $graph->addItem(
                $dbGraphItem['itemid'],
                $dbGraphItem['yaxisside'],
                $dbGraphItem['calc_fnc'],
                $dbGraphItem['color'],
                $dbGraphItem['drawtype'],
                $dbGraphItem['type'],
                $dbGraphItem
        );
}

3、保留is_history数据

public function addItem($itemid, $axis = GRAPH_YAXIS_SIDE_DEFAULT, $calc_fnc = CALC_FNC_AVG, $color = null, $drawtype = null, $type = null, $item_history = null) {

$item['is_history'] = $item_history['is_history'];

4、处理is_history画图

                                if($this->items[$i]['is_history'])
                                array_push($sql_arr,
                                        'SELECT itemid,'.$calc_field.' AS i,'.
                                                'COUNT(*) AS count,AVG(value) AS avg,MIN(value) as min,'.
                                                'MAX(value) AS max,MAX(clock) AS clock'.
                                        ' FROM history '.
                                        ' WHERE itemid='.zbx_dbstr($this->items[$i]['itemid']).
                                                ' AND clock>='.zbx_dbstr($from_time-$this->items[$i]['is_history']*86400).
                                                ' AND clock<='.zbx_dbstr($to_time-$this->items[$i]['is_history']*86400).
                                        ' GROUP BY itemid,'.$calc_field
                                        ,
                                        'SELECT itemid,'.$calc_field.' AS i,'.
                                                'COUNT(*) AS count,AVG(value) AS avg,MIN(value) AS min,'.
                                                'MAX(value) AS max,MAX(clock) AS clock'.
                                        ' FROM history_uint '.
                                        ' WHERE itemid='.zbx_dbstr($this->items[$i]['itemid']).
                                                ' AND clock>='.zbx_dbstr($from_time-$this->items[$i]['is_history']*86400).
                                                ' AND clock<='.zbx_dbstr($to_time-$this->items[$i]['is_history']*86400).
                                        ' GROUP BY itemid,'.$calc_field
                                );
                                else


未完。。。
