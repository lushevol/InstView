本组件致力于快速搭建业务系统，减少代码重复的编写，用更少的时间做更多的事情。

- InnerForm

快速form表单，输入初始数据和表单列属性，即可生成form表单。

 **1. referTable: 表单列属性**
 
 ```json
 {
 	  label,             // 文本
     prop,              // 属性名
   	selectProp,   // 下拉框属性名
   	submitProp,   // 提交时的属性名
   	editable,       // 是否可编辑
   	tags,               // {Object} 有值时，可增加。包含字段和数据类型
   	refer,             // 显示的值的对照表，{Array} 值和显示文字的对照表[{label: 'value', value: 'key'}]，{Function({ rawVal })} return 显示的文字, {Object} key-value 对照
   	formHidden,       // 是否隐藏
   	type,         	// value的数据类型，类型如下：
 					// [text] 文本,
 					// [textarea]
 					// [year/month/date/dates/week/datetime/datetimerange/daterange/monthrange] 日期
 					// [time] 时间
   	format,           // 规定value的格式，仅部分type生效
   	rows,               // textarea rows 属性
   	selectOptions            // select的选项
 } 
 ```

- InstantTable

  快速table表格
