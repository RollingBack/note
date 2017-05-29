===========================
Python
===========================

---------------------------
Python的100个坑
---------------------------

1.缩进语法
============================

典型双刃剑。

2. 试图往列表中插入数据，不是不可以，而是效率低。
====================================================================================
::

	def asc_list(j):
	    nums = []
	    for i in range(j):
	        nums.append(i)
	    return nums.reverse()

	def desc_list(j):
	    nums = []
	    for i in range(j):
	        nums.insert(0, i)
	    return nums

在IPython中执行的效率对比
::

	In [29]: count = 10**5

	In [30]: %timeit asc_list(count)
	100 loops, best of 3: 11.2 ms per loop
	
	In [31]: %timeit desc_list(count)
	1 loops, best of 3: 3.12 s per loop

3. Python不是弱类型语言,php才是。
=========================================================

.. code-block:: sh

	In[2]: myint = 1
	In[3]: mystring = '1'
	In[4]: myint + mystring
    #TypeError: unsupported operand type(s) for +: 'int' and 'str'

----------------------------------------------------
抓站代码，请慎用，尊重别人的劳动
----------------------------------------------------

.. code-block:: python

		# -*- coding: utf-8 -*-
	"""
	Spyder Editor

	This is a temporary script file.
	"""

	import HTMLParser

	class myHTMLparser(HTMLParser.HTMLParser):
	    def __init__(self):
	        HTMLParser.HTMLParser.__init__(self)
	        self.flag = 0
	        self.begin = 0
	        self.count = 0
	        self.links = []
	        
	    def handle_startendtag(self, tag, attrs):
	        if tag == 'img':            
	            for (attr, value) in attrs:
	                if(attr == 'src'):
	                    self.begin += 1
	                    if(self.begin > 2):
	                        self.links.append(value.split('/')[-1])
	            self.flag = 1
	        
	    def handle_starttag(self, tag, attrs):
	        if tag == 'a' and self.begin > 2 and self.begin < 95:
	            self.links.append(attrs[0][1].split('/')[-1])
	        elif self.begin == 95:
	            self.links.append(attrs[0][1].split('/')[-1])
	            self.begin += 1
	        else:
	            pass
	        self.count += 1
	            
	      
	    def handle_data(self, data):
	        if self.flag == 1:
	            self.links.append(data)
	            self.flag = 0
	            
	    
	    
	        
	def file_get_content(path):
	    f = open(path, 'r')
	    string = f.read()
	    f.close
	    return string

	html = file_get_content('./3dmisc.html')
	parser = myHTMLparser()
	parser.feed(html)
	mylist = parser.links
	mylist = mylist[2:]
	new_list = []
	mylistlen = len(mylist)
	page = mylistlen/3
	f = open('./1.sql', 'w+')
	for i in range(0, page-1):
	    f.write('INSERT INTO ')
	f.close()

