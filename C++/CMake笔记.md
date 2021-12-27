## include_directories作用域  
include_directories仅作用于当前目录和子目录，不能回传到父目录

## 链接问题
add_library如果添加静态库不需要target_link_libraries，动态库才需要；  
target_link_libraries链接顺序是有要求的，排在前面的库可以依赖排在后面的库，反之会报错  
如果存在循环依赖的问题（比如A、B两个库互相依赖），有一个骚操作可以解决，把所有需要链接的库的名字存成一个list：
```
set(LINKED_LIBS A B)
target_link_libraries(target ${LINKED_LIBS} ${LINKED_LIBS})
```
将link列表添加两次，可以正常编译通过，但不知道是否会有坑