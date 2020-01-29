# 什麼是 AU
AU 代表 Asynchornous Update，是指 ZK 從瀏覽器發出 AJAX 並局部更新頁面的過程。 

# AU request
當事件在瀏覽器被觸發，ZK 元件會發出 AU request，透過開發者工具觀察 Form Data 可得知事件的細節，其中包含以下欄位：

* `dtid`, Desktop id
* `cmd`, 事件名稱, 例如 `onClick`
* `uuid`, 發生該事件的 DOM element id
* `data`, 事件相關資料，每個事件會帶不同的資料


# AU response

## 重繪元件
```
{"rs":[["outer",[{$u:'yZ6Q2'},[
['zul.tab.Tabbox','yZ6Q2',{width:'20%',prolog:'\n\t'},{},[
['zul.tab.Tabs','yZ6Q3',{},{},[
['zul.tab.Tab','yZ6Q8',{$onSelect:false,$onClose:true,label:'tab 1’,iconSclass:'z-icon-book',selected:true},{},[]],
['zul.tab.Tab','yZ6Qf',{$onSelect:false,$onClose:true,label:'tab 2’,iconSclass:'z-icon-book'},{},[]]]],
…],"rid":1}
```

## 增加子元件
```
{"rs":[["addChd",["iB0Zk0",[
['zul.sel.Treeitem','iB0Zy1',{open:false,_loaded:true},{},[
['zul.sel.Treerow','iB0Zz1',{},{},[
['zul.sel.Treecell','iB0Z_2',{label:'< 2.0L(inc.)’},{},[]],
['zul.sel.Treecell','iB0Z02',{label:'3'},{},[]]]]]]],…,"rid":7}
```

## 刪除元件
```
{"rs":[["rm",["zY5Q10"]],["rm",["zY5Qz"]],["rm",["zY5Q60"]],["rm",["zY5Q80"]],["rm",["zY5Qa0"]],["rm",["zY5Qe0"]],["rm",["zY5Qc0"]],["rm",["zY5Qq0"]],["rm",["zY5Qo0"]],["rm",["zY5Qm0"]],…}
```
