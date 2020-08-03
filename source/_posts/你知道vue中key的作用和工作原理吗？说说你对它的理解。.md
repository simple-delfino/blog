---
title: 你知道vue中key的作用和工作原理吗？说说你对它的理解。
id: '20200729002'
tags:
  - interview
date: 2020-07-29 14:19:58
tag: interview
category: web
---

先写测试案例代码

```
<body>
 <div id="demo">
 <p v-for="item in items" :key="item">{{item}}</p>
 </div>
 <script src="../dist/vue.js"></script>
 <script>
 //创建实例
 const app = new Vue({
 el: '#demo',
 data:{items:['a','b','c','d','e']},
 mounted(){
 setTimeout(() => {
 this.items.splice(2,0,'f')//把c替换为f
 }, 2000);
 }
 })
 </script>
</body>
```

案例解析

###### 先简单介绍一下虚拟DOM的Diff算法

vue和react的虚拟DOM的Diff算法大致相同，其核心是基于两个简单的假设：

*   两个相同的组件产生类似的DOM结构，不同的组件产生不同的DOM结构。
    
*   同一层级的一组节点，他们可以通过唯一的id进行区分。
    

基于以上这两点假设，使得虚拟DOM的Diff算法的复杂度从O(n^3)降到了O(n)。这里我们借用React’s diff algorithm中的一张图来简单说明一下：

![](https://imgkr.cn-bj.ufileos.com/a008ec4e-a3b5-4bcc-a79f-2b354ec91a20.png)

当页面的数据发生变化时，Diff算法只会比较同一层级的节点：

*   如果节点类型不同，直接干掉前面的节点，再创建并插入新的节点，不会再比较这个节点以后的子节点了。
    
*   如果节点类型相同，则会重新设置该节点的属性，从而实现节点的更新。
    

当某一层有很多相同的节点时，也就是列表节点时，Diff算法的更新过程默认情况下也是遵循以上原则。

分析测试代码，我们希望可以在B和C之间加一个F

![](https://imgkr.cn-bj.ufileos.com/a9e3d61c-e8be-4b20-bd00-19fe7c2b0030.png)

不加key时，Diff算法默认执行起来是这样的：

![](https://imgkr.cn-bj.ufileos.com/2003083e-e308-4f59-b975-c0c7252c5fe3.png) 即把C更新成F，D更新成C，E更新成D，最后再插入E，是不是很没有效率？

加上key时，使用key来给每个节点做一个唯一标识，Diff算法就可以正确的识别此节点，找到正确的位置区插入新的节点。

![](https://imgkr.cn-bj.ufileos.com/cc59fe40-2136-43b1-b69e-4e16f47df4a2.png)

```
// 首次循环patch A A B C D E 
A B F C D E 
 // 第2次循环patch B B C D E 
B F C D E 
 // 第3次循环patch E C D E 
F C D E 
 // 第4次循环patch D C D 
F C D 
 // 第5次循环patch C C  
F C 
 // oldCh全部处理结束，newCh中剩下的F，创建F并插入到C前面
 ``` 

源码分析（执行测试代码进行调试）

头尾比较4种情况 :新节点从头开始和旧节点从头开始，新头和旧尾，旧头和新尾，旧头和旧尾

### 源码

源码在patch的过程中，会执行patchVnode，patchVnode过程中会执行updateChildren这个方法，他会更新所有的两个新旧的子元素，那么在这个过程中，通过key就可以精准的判断，当前在循环的这两个节点是不是一个节点。

```
//循环条件，开始索引不能大于结束索引
while (oldStartIdx <= oldEndIdx && newStartIdx <= newEndIdx) {
//头尾指针调整
 if (isUndef(oldStartVnode)) {
 oldStartVnode = oldCh[++oldStartIdx] // Vnode has been moved left
 } else if (isUndef(oldEndVnode)) {
 oldEndVnode = oldCh[--oldEndIdx]
//后面是头尾比较4中情况
 } else if (sameVnode(oldStartVnode, newStartVnode)) {
 patchVnode(oldStartVnode, newStartVnode, insertedVnodeQueue, newCh, newStartIdx)
 ...
 } else if (sameVnode(oldEndVnode, newEndVnode)) {
//两个开头相同
 patchVnode(oldEndVnode, newEndVnode, insertedVnodeQueue, newCh, newEndIdx)
//索引向后移动一位
 oldStartVnode = oldCh[++oldStartIdx]
 newStartVnode = newCh[++newStartIdx]
 } else if (sameVnode(oldStartVnode, newEndVnode)) { // Vnode moved right
 patchVnode(oldStartVnode, newEndVnode, insertedVnodeQueue, newCh, newEndIdx)
 ...
 } else if (sameVnode(oldEndVnode, newStartVnode)) { // Vnode moved left
 patchVnode(oldEndVnode, newStartVnode, insertedVnodeQueue, newCh, newStartIdx)
 ...
 } else {
 ...
 }
//整理工作:必定有数组还有剩下的元素未处理
 if (oldStartIdx > oldEndIdx) {
//老的结束了，这种情况说明新的数组里还有剩下的节点
 refElm = isUndef(newCh[newEndIdx + 1]) ? null : newCh[newEndIdx + 1].elm
 addVnodes(parentElm, refElm, newCh, newStartIdx, newEndIdx, insertedVnodeQueue)
 } else if (newStartIdx > newEndIdx) {
//新的结束了，此时删除老数组中剩下的即可
 removeVnodes(parentElm, oldCh, oldStartIdx, oldEndIdx)
 }
 ``` 

### 不加key

如果不加key，每一次都会判断为相同节点，那么就会进入patchVnode方法，强行进行更新。判断相同节点的方法是sameVnode

```
function sameVnode (a, b) {
 return (
 a.key === b.key && (
 (
 a.tag === b.tag &&//标签名
 a.isComment === b.isComment &&//是否为注释
 isDef(a.data) === isDef(b.data) &&//data有没有发生变化
 sameInputType(a, b)//是不是input类型
 ) || (
 isTrue(a.isAsyncPlaceholder) &&
 a.asyncFactory === b.asyncFactory &&
 isUndef(b.asyncFactory.error)
 )
 )
 )
}
``` 

### 加了key

如果加上key，如果新旧节点不是相同节点就不会进入patchVnode方法

结论
--

1.  key的作用主要是为了高效的更新虚拟DOM，其原理是vue在patch过程中通过key可以精准判断两 个节点是否是同一个，从而避免频繁更新不同元素，使得整个patch过程更加高效，减少DOM操 作量，提高性能。
    
2.  另外，若不设置key还可能在列表更新时引发一些隐蔽的bug
    
3.  vue中在使用相同标签名元素的过渡切换时，也会使用到key属性，其目的也是为了让vue可以区分 它们，否则vue只会替换其内部属性而不会触发过渡效果。
    
