---
title: form 表单提交时阻止页面跳转
comments: false
---
## 问题
使用表单提交文件下载请求时，会造成页面跳转。使用如下方法可在表单提交时阻止页面跳转。
## 方案
```html
<html>
<body>
	<form action="" method="post" target="nm_iframe">
		<input type="text" id="id_input_text" name="nm_input_text" />
		<input type="submit" id="id_submit" name="nm_submit" value="提交" />
	</form>
	<iframe id="id_iframe" name="nm_iframe" style="display:none;"></iframe>
</body>
</html>
```