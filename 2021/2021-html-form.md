form 表单

action： 控制数据后端的提交路径

 	1. 空 null 什么都不写  默认超当前页面提交url数据
 	2. 写全路径： http://www.baidu.com    指定路径
 	3. 只写后端后缀路径action=/index/   自动识别ip:port/index/ 拼接

```html
<h1> 标题标签 </h1>
<form action="">
  <p>
    <label for="d1">
         username: <input id="d1" type="text">
    </label>
  </p>
    <label for="d2">
        password
    </label>
    <input type="text", id="d2">
</form>

"""
label intut 都是行内标签
"""

input 标签 通过type属性变形
	text 普通文本
  password 密文 
  data  日期
  submit: 用来出发一个提交数据的动作
  button： 普普通通的按钮，可以自己根据js 绑定各种功能
  reset： 重置内容
  radio:  单选


## 能够出发form 表单提交数据的按钮有哪些 
1 <input type='sumit' value="zhuce"> 
2 <button> dian wo  </button>


```



form 表单提交文件需要注意

​	method 必须是post 

​	enctype = "multipart/form-data"

​    enctype  类似于数据提交的编码格式 改成formdata 支持可以提交文件数据

​    file_obj = request.files.get('myfile')

​    File_obj.save(file_obj.name)





针对用户输入的标签 如果加了value 就是默认值

disable：禁用

readonly： 只读

hidden： 隐藏框

