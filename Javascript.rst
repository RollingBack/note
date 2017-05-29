================================
HTML CSS Javascript Jquery
================================


-------------------------------------
Javascript的100个坑
-------------------------------------

**1.** 不是专业前端，经常写PHP，免不了要写出下面这样的代码：

.. code-block:: javascript

	function a(b=0){
		
	}

默认参数貌似FireFox是支持的，但是Chrome下就会报错。最好写成这样：

.. code-block:: javascript

	function a(b){
		if(b=='' || b==null){
			b = 0;
		}
	}

**2.** 如何判定undefine

.. code-block:: javascript
	
	if(typeof a == 'undefined'){

	}

**3.** eval('('+jsondata+')')报错
记得防御编程，jsondata为空时就会报错。