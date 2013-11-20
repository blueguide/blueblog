---
layout : website
title : Bootstrap-wysiwyg.js - Source Code Analysis
categories : ['code','source code analysis']
---

<h1>{{ page.title}}</h1>

***

### Introduce

The [bootstrap-wysiwyg](http://mindmup.github.io/bootstrap-wysiwyg/) is a tiny JQuery Bootstrap plugin.

### How to read

In order to facilitate reading the source code comments, please refer to the following agreement:


### Source code analysis

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
	
	/*The main body*/
	$.fn.wysiwyg = function (userOptions) { 
		var editor = this,
				selectedRange,
				options,//The config options
				toolbarBtnSelector,
				/*To update the tool bar's display.By adding/removing the style(activeToolbarClass), change the toolbar's display.*/
				updateToolbar = function () {
					//To judge whether the activeToolbarClass is defined
					if (options.activeToolbarClass) {
						//Traversing toolbar
						$(options.toolbarSelector).find(toolbarBtnSelector).each(function () {
							//To get the command of the tool
							var command = $(this).data(options.commandRole);
							//
							if (document.queryCommandState(command)) {
								$(this).addClass(options.activeToolbarClass);
							} else {
								//
								$(this).removeClass(options.activeToolbarClass);
							}
						});
					}
				},
				execCommand = function (commandWithArgs, valueArg) {
					var commandArr = commandWithArgs.split(' '),
						command = commandArr.shift(),
						args = commandArr.join(' ') + (valueArg || '');
					document.execCommand(command, 0, args);
					updateToolbar();
				},
				/*bind hot keys function*/
				bindHotkeys = function (hotKeys) {//**********(6)*********
					//iterate through the config options 
					$.each(hotKeys, function (hotkey, command) {
						//listen to the keydown and keyup event to perform the action with "exeCommand"
						editor.keydown(hotkey, function (e) {
							if (editor.attr('contenteditable') && editor.is(':visible')) {
								e.preventDefault();
								e.stopPropagation();
								execCommand(command);
							}
						}).keyup(hotkey, function (e) {
							if (editor.attr('contenteditable') && editor.is(':visible')) {
								e.preventDefault();
								e.stopPropagation();
							}
						});
					});
				},
				getCurrentRange = function () {
					var sel = window.getSelection();
					if (sel.getRangeAt && sel.rangeCount) {
						return sel.getRangeAt(0);
					}
				},
				saveSelection = function () {
					selectedRange = getCurrentRange();
				},
				restoreSelection = function () {
					var selection = window.getSelection();
					if (selectedRange) {
						try {
							selection.removeAllRanges();
						} catch (ex) {
							document.body.createTextRange().select();
							document.selection.empty();
						}

						selection.addRange(selectedRange);
					}
				},
				insertFiles = function (files) {
					editor.focus();
					$.each(files, function (idx, fileInfo) {
						if (/^image\//.test(fileInfo.type)) {
							$.when(readFileIntoDataUrl(fileInfo)).done(function (dataUrl) {
								execCommand('insertimage', dataUrl);
							}).fail(function (e) {
								options.fileUploadError("file-reader", e);
							});
						} else {
							options.fileUploadError("unsupported-file-type", fileInfo.type);
						}
					});
				},
				markSelection = function (input, color) {
					restoreSelection();
					if (document.queryCommandSupported('hiliteColor')) {
						document.execCommand('hiliteColor', 0, color || 'transparent');
					}
					saveSelection();
					input.data(options.selectionMarker, color);
				},
				bindToolbar = function (toolbar, options) {
					toolbar.find(toolbarBtnSelector).click(function () {
						restoreSelection();
						editor.focus();
						execCommand($(this).data(options.commandRole));
						saveSelection();
					});
					toolbar.find('[data-toggle=dropdown]').click(restoreSelection);

					toolbar.find('input[type=text][data-' + options.commandRole + ']').on('webkitspeechchange change', function () {
						var newValue = this.value; /* ugly but prevents fake double-calls due to selection restoration */
						this.value = '';
						restoreSelection();
						if (newValue) {
							editor.focus();
							execCommand($(this).data(options.commandRole), newValue);
						}
						saveSelection();
					}).on('focus', function () {
						var input = $(this);
						if (!input.data(options.selectionMarker)) {
							markSelection(input, options.selectionColor);
							input.focus();
						}
					}).on('blur', function () {
						var input = $(this);
						if (input.data(options.selectionMarker)) {
							markSelection(input, false);
						}
					});
					toolbar.find('input[type=file][data-' + options.commandRole + ']').change(function () {
						restoreSelection();
						if (this.type === 'file' && this.files && this.files.length > 0) {
							insertFiles(this.files);
						}
						saveSelection();
						this.value = '';
					});
				},
				initFileDrops = function () {
					editor.on('dragenter dragover', false)
						.on('drop', function (e) {
							var dataTransfer = e.originalEvent.dataTransfer;
							e.stopPropagation();
							e.preventDefault();
							if (dataTransfer && dataTransfer.files && dataTransfer.files.length > 0) {
								insertFiles(dataTransfer.files);
							}
						});
				};
		//The config options:merge the content of user's config and default config
		options = $.extend({}, $.fn.wysiwyg.defaults, userOptions);
		
		toolbarBtnSelector = 'a[data-' + options.commandRole + '],button[data-' + options.commandRole + '],input[type=button][data-' + options.commandRole + ']';
		
		//bind hot keys
		bindHotkeys(options.hotKeys);
		
		//make the div editable
		//'contenteditable' is a css 3 attribute.
		//"Sets or retrieves the string that indicates whether the user can edit the content of the object."
		editor.attr('contenteditable', true)
			//listen to mouseup,keyup and mouseout event
			.on('mouseup keyup mouseout', function () {
				//
				saveSelection();
				//update the toolbars style
				updateToolbar();
			});
		return this;
	}
	
	/*The default config*/
	$.fn.wysiwyg.defaults = {
		hotKeys: {
			'ctrl+b meta+b': 'bold',
			'ctrl+i meta+i': 'italic',
			'ctrl+u meta+u': 'underline',
			'ctrl+z meta+z': 'undo',
			'ctrl+y meta+y meta+shift+z': 'redo',
			'ctrl+l meta+l': 'justifyleft',
			'ctrl+r meta+r': 'justifyright',
			'ctrl+e meta+e': 'justifycenter',
			'ctrl+j meta+j': 'justifyfull',
			'shift+tab': 'outdent',
			'tab': 'indent'
		},
		toolbarSelector: '[data-role=editor-toolbar]',
		commandRole: 'edit',
		activeToolbarClass: 'btn-info',
		selectionMarker: 'edit-focus-marker',
		selectionColor: 'darkgrey',
		dragAndDropImages: true,
		fileUploadError: function (reason, detail) { console.log("File upload error", reason, detail); }
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
-	[css3 'contenteditable'](http://www.w3school.com.cn/html5/att_global_contenteditable.asp)
-	[CONTENTEDITABLE Attribute | contentEditable Property](http://msdn.microsoft.com/en-us/library/ms537837)
-	[The Road to HTML 5: contentEditable](http://blog.whatwg.org/the-road-to-html-5-contenteditable)
