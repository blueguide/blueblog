---
layout : website
title : Bootstrap-wysiwyg.js - Source code analysis
categories : ['code','source code analysis']
---

<h1>{{ page.title}}</h1>

***

Source code analysis

{% highlight JavaScript linenos %}
(function ($) {
	'use strict';//Strict Mode (JavaScript)
	/*Get the HTML contents*/
	$.fn.cleanHtml = function () {
		//Get the HTML contents
		var html = $(this).html();
		//use the regular expression to filter the <br>,&nbsp and empty <div> tags.
		//&&:A&&B,if A=true return B else return A; A=null or undefined or '',then return '';
		//the key feature:(Optionally) cleans up trailing whitespace and empty divs and spans
		return html && html.replace(/(<br>|\s|<div><br><\/div>|&nbsp;)*$/, '');
	};
	
	
}(window.jQuery));
{% endhighlight %}

	
related reading
---------------

-	[jslint](http://www.jslint.com/lint.html)
-	[use strict-Strict Mode(cn)](http://qianduan-notes.diandian.com/post/2012-06-02/40027620460)
-	[use strict-Strict Mode(en)](http://msdn.microsoft.com/en-us/library/ie/br230269%28v=vs.94%29.aspx)
-	[jquery $.fn $.fx](http://hi.baidu.com/jjjvzugcpmcdmor/item/0e32a89c36a18544f04215d7)
-	[The 30 Minute Regex Tutorial(cn)](http://www.cnblogs.com/deerchao/archive/2006/08/24/zhengzhe30fengzhongjiaocheng.html)
-	[The 30 Minute Regex Tutorial(en)](http://www.codeproject.com/Articles/9099/The-30-Minute-Regex-Tutorial)
-	[javascript '||' and '&&'](http://my249645546.iteye.com/blog/1553202)
-	[JavaScript 'use strict'](http://qianduan-notes.diandian.com/post/2012-06-02/40027620460)
-	[JavaScript '#.extend()'(cn)](http://www.cnblogs.com/RascallySnake/archive/2010/05/07/1729563.html)
-	[JavaScript '#.extend()'(en)](http://api.jquery.com/jQuery.extend/)
-	[execCommand compatibility](http://www.quirksmode.org/dom/execCommand.html)
-	[JavaScript '.data()'](http://api.jquery.com/data/)
-	[queryCommandState](http://www.hbcms.com/main/dhtml/methods/querycommandstate.html)
-	[Jquery '$.each()' '$().each()'](http://blog.csdn.net/on_my_way20xx/article/details/7791769)
-	[Javascript document.execCommand()](http://zhaosheng.wolf.blog.163.com/blog/static/115304589200992615215759/?fromdm&fromSearch&isFromSearchEngine=yesdocument.execCommand)
-	[Javascript '.keydown()'](http://api.jquery.com/keydown/)