# 文章链接：[foreach 中使用引用的问题](https://my.oschina.net/agui1989/blog/1606082)

php 通过循环修改一个数组是常用的操作，可以通过这种方法实现：

	foreach($arr as $k=>$v){

	$arr[$k]=$v*2;

	}
这种方式直接操作数据很简洁，也不容易带来其它负作用，但很多时候我们并没有用到$k，于是就会跳过$k，直接使用“引用”：

	foreach($arr as &$v){

	$v=$v*2;

	}
一般情况下，我们这样使用并不会带来什么问题，但有这么两种情况会出现意想不到的结果，尽管现实的业务代码中不太可能出现，但了解一下总是好的：

情况一：

	$arr=array(1,2,3);
	foreach($arr as &$v){

	}
	foreach($arr as $v){

	}
	var_dump($arr);

//结果：array(3) { [0]=> int(1) [1]=> int(2) [2]=> &int(2) }
这种情况的结果：

如果多试几个数组的话，你会发现：数组的最后一个元素被修改为倒数第二个元素了。

 

发生的条件：

1. 在同一个作用域下同一个数组循环了2次（循环多次也会出现类似的结果）；

2. 第一个循环中的$v使用了引用，第二个没有使用引用

如果这两个条件不能同时满足，就不会出现这种“异常”。

 

为什么会出现这种情况呢？

原因有两点：

1. 每次foreach循环的过程，实际上有个隐形的操作：

	foreach($arr as $k=>$v){

	$v=$arr[$k]; //内部实际有这一步的操作，即使你没有写$k,内部也有个临时变量，作用同$k

	}
2. 当$v使用引用的时候，循环完成后$v保留了对数据最后一个元素，$arr[$k] 的引用。

即第一次循环后，$v=$arr[2];

第二次循环的时候：

	foreach($arr as $v){
	//$k=0时: 
		$v=$arr[0];//因为$v是$arr[2]的引用，所以这一步相当于：$arr[2]=$arr[0];
	//$k=1 时：
		$arr[2]=$arr[1];
	//$k=2 时：
		$arr[2]=$arr[2];//而$arr[2]，在当$k=1时赋值为什么$arr[1]，所以最后$arr[2]=$arr[1]
	}
 

 

情况二：

	$arr=array(1,2,3);
	foreach($arr as $k=>$v){
		$v=&$arr[$k];
	}
	var_dump($arr);
//结果：array(3) { [0]=> int(2) [1]=> int(3) [2]=> &int(3) }
这种情况的结果是：

数组的每个元素左移一位，最后一位不变。

 

为什么会这样呢？

如果了解前一种情况的话，这种情况理解起来就比较容易了：

	$arr=array(1,2,3);
	foreach($arr as $k=>$v){
		$v=&$arr[$k];
		/* 
		当 $k=0 时：
		$v=&$arr[0];
		当 $k=1 时：
		$v=&$arr[1];而$v而是$arr[0]的引用，即：$arr[0]=&$arr[1];
		当 $k=2 时：
		$v=&$arr[2];即：$arr[1]= &$arr[2];因为每一次赋值的操作对象都是前一个元素，所以最后一个元素并没有被左移
		*/
	}