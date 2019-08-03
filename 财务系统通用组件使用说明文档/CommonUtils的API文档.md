@[TOC](CommonUtils.js的使用方法)

# 1. 引入CommonUtils.js文件

* Note：相对路径需要根据自己的文件进行修改！
```html
<script type="text/javascript" src="../../Utils/CommonUtils.js"></script>
```

# 2. 包含功能：

* easyui datagrid 导出excel文件功能

# 3. 使用方法：

```js
// 将id = main_table的datagrid中的数据导出为excel文件
$('#main_table').datagrid('toExcel',"凭证流水查询表");
```

# 4. CommonUtils.js源代码

```javascript
// 打印功能
function CreateFormPage(strPrintName, printDatagrid,headerNoDataNum) {
    // var tableTab = document.getElementsByTagName("table");
    // tableTab.parentNode.appendChild("<iframe id='printf' src='' width='0' height='0' frameborder='0'></iframe>");
    document.write("<iframe id='printf' src='' width='0' height='0' frameborder='0'></iframe>");
    var tableString = '<table cellspacing="0" class="pb" width="100%" border="1em">';
    // 得到frozenColumns对象
    var frozenColumns = printDatagrid.datagrid("options").frozenColumns;
    // 得到columns对象
    var columns = printDatagrid.datagrid("options").columns;
    var nameList = '';

    // 载入title
    if (typeof columns != 'undefined' && columns != '') {
        $(columns).each(function (index) {
            tableString += '\n<tr>';
            if (typeof frozenColumns != 'undefined' && typeof frozenColumns[index] != 'undefined') {
                for (var i = 0; i < frozenColumns[index].length; ++i) {
                    if (!frozenColumns[index][i].hidden) {
                        tableString += '\n<th width="' + frozenColumns[index][i].width + '"';
                        if (typeof frozenColumns[index][i].rowspan != 'undefined' && frozenColumns[index][i].rowspan > 1) {
                            tableString += ' rowspan="' + frozenColumns[index][i].rowspan + '"';
                        }
                        if (typeof frozenColumns[index][i].colspan != 'undefined' && frozenColumns[index][i].colspan > 1) {
                            tableString += ' colspan="' + frozenColumns[index][i].colspan + '"';
                        }
                        if (typeof frozenColumns[index][i].field != 'undefined' && frozenColumns[index][i].field != '') {
                            nameList += ',{"f":"' + frozenColumns[index][i].field + '", "a":"' + frozenColumns[index][i].align + '"}';
                        }
                        tableString += '>' + frozenColumns[0][i].title + '</th>';
                    }
                }
            }
            for (var i = 0; i < columns[index].length; ++i) {
                if (!columns[index][i].hidden) {
                    tableString += '\n<th width="' + columns[index][i].width + '"';
                    if (typeof columns[index][i].rowspan != 'undefined' && columns[index][i].rowspan > 1) {
                        tableString += ' rowspan="' + columns[index][i].rowspan + '"';
                    }
                    if (typeof columns[index][i].colspan != 'undefined' && columns[index][i].colspan > 1) {
                        tableString += ' colspan="' + columns[index][i].colspan + '"';
                    }
                    if (typeof columns[index][i].align != 'undefined') {
                        tableString += ' align="' + columns[index][i].align + '"';
                    }
                    if (typeof columns[index][i].field != 'undefined' && columns[index][i].field != '') {
                        nameList += ',{"f":"' + columns[index][i].field + '", "a":"' + columns[index][i].align + '"}';
                    }
                    tableString += '>' + columns[index][i].title + '</th>';
                }
            }
            tableString += '\n</tr>';
        });
    }
    // 载入内容
    var rows = printDatagrid.datagrid("getRows"); // 这段代码是获取当前页的所有行
    var nl = eval('([' + nameList.substring(1) + '])');
    nl.splice(0,headerNoDataNum);
    for (var i = 0; i < rows.length; ++i) {
        tableString += '\n<tr>';
        $(nl).each(function (j) {
            var e = nl[j].f.lastIndexOf('_0');

            tableString += '\n<td';
            if (nl[j].a != 'undefined' && nl[j].a != '') {
                tableString += ' style="text-align:' + nl[j].a + ';"';
            }
            tableString += '>';
            if (e + 2 == nl[j].f.length) {
                tableString += rows[i][nl[j].f.substring(0, e)];
            }
            else{
                //此处有更改，设置成如果读取字段为null，设置自动填充0
                if(rows[i][nl[j].f] == null){
                    if(i != rows.length-1)
                        tableString += 0;
                }else {
                    tableString += rows[i][nl[j].f];
                }
            }
            tableString += '</td>';
        });
        tableString += '\n</tr>';
    }
    tableString += '\n</table>';
    // 生成并打印ifrme
    var f = document.getElementById('printf');
    f.contentDocument.write(tableString);
    f.contentDocument.title = strPrintName;
    f.contentDocument.close();
    f.contentWindow.print();
}
//自定义导出excel表格
var commonUtils = function () {
    $.extend($.fn.datagrid.methods, {
        toExcel: function(jq, filename){
            return jq.each(function(){
                var uri = 'data:application/vnd.ms-excel;base64,'
                    , template = '<html xmlns:o="urn:schemas-microsoft-com:office:office" xmlns:x="urn:schemas-microsoft-com:office:excel" xmlns="http://www.w3.org/TR/REC-html40"><head><!--[if gte mso 9]><xml><x:ExcelWorkbook><x:ExcelWorksheets><x:ExcelWorksheet><x:Name>{worksheet}</x:Name><x:WorksheetOptions><x:DisplayGridlines/></x:WorksheetOptions></x:ExcelWorksheet></x:ExcelWorksheets></x:ExcelWorkbook></xml><![endif]--></head><body><table border="1" cellspacing="0">{table}</table></body></html>'
                    , base64 = function (s) { return window.btoa(unescape(encodeURIComponent(s))) }
                    , format = function (s, c) { return s.replace(/{(\w+)}/g, function (m, p) { return c[p]; }) }

                var alink = $('<a style="display:none"></a>').appendTo('body');
                var view = $(this).datagrid('getPanel').find('div.datagrid-view');

                var table = view.find('div.datagrid-view2 table.datagrid-btable').clone();
                var tbody = table.find('>tbody');
                view.find('div.datagrid-view1 table.datagrid-btable>tbody>tr').each(function(index){
                    $(this).clone().children().prependTo(tbody.children('tr:eq('+index+')'));
                });

                var head = view.find('div.datagrid-view2 table.datagrid-htable').clone();
                var hbody = head.find('>tbody');
                view.find('div.datagrid-view1 table.datagrid-htable>tbody>tr').each(function(index){
                    $(this).clone().children().prependTo(hbody.children('tr:eq('+index+')'));
                });
                hbody.prependTo(table);

                var ctx = { worksheet: name || 'Worksheet', table: table.html()||'' };
                alink[0].href = uri + base64(format(template, ctx));
                alink[0].download = filename;
                alink[0].click();
                alink.remove();
            })
        }
    });
};
$.fn.window.defaults.onOpen = commonUtils;
```

