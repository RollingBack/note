=========================================
flask
=========================================


---------------------------------------
一些坑
---------------------------------------

**1.** 加入路由的函数总是需要返回String:
下面代码会报错，TypeError: 'float' object is not callable

.. code-block:: python

    @app.route('/calculate', methods=['POST'])
    def calculate():
        b = float('0.8')
        return b

