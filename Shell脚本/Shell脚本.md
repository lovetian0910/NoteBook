## if [-z ${version+x}]问题

这行脚本的本意是判断version变量是否有设置  
* shell脚本中，if[-z string]代表判断字符串是否为空
* ${var+x}表示如果变量var有设置，则返回+后面的字符，这里x可以为任意字符；但是如果var没有设置，则返回空  

经过实验发现如果不加上+x，当version设置为空字符串，或者开头为空格的字符串，这个if语句都会返回true；  
而加上+x，只有当version没有定义的时候，才会返回true。