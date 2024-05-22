# vue2-virtual-list

虚拟列表在我们渲染大量列表数据时，可以用到来优化渲染达到减少渲染时间的作用。
<a name="X5H3n"></a>

# 什么是虚拟列表

虚拟列表其实是按需显示的一种实现**，即只对可见区域进行渲染，对非可见区域中的数据不渲染或部分渲染的技术**，从而达到极高的渲染性能。
<a name="xqaVH"></a>

# 定高虚拟列表

<a name="DKa8S"></a>

### 数据的解决

 关于定高虚拟列表的实现是为了解决在列表中渲染大量数据造成加载缓慢的问题的。我们首先准备一个包含大量数据的数组来模拟实际情况。<br />![屏幕截图 2024-05-22 203734.png](https://cdn.nlark.com/yuque/0/2024/png/40660095/1716381445279-429c6f6a-8fd1-44bd-b378-bbfafda926b6.png#averageHue=%23252830&clientId=u25dc4089-44af-4&from=ui&id=u49023f38&originHeight=281&originWidth=1275&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=50120&status=done&style=none&taskId=ubfbc09ef-8b88-4885-abbd-3ebb9ef700f&title=)<br />我们将数组传入到子组件中，在子组件中我们通过props来接受，同时被接受的还有每个列表项的高度（定高）。
<a name="zCx0j"></a>

### 关键的信息

由于我们需要对可见部分进行渲染，所以我们需要一些关键的变量来进行计算，比如说可见部分列表项的开始索引，结束索引，可视区域的高度，偏移量（这个尤其重要）<br />那么，就让我们来看一下，这些变量是如何计算出来的：<br />![屏幕截图 2024-05-22 204624.png](https://cdn.nlark.com/yuque/0/2024/png/40660095/1716381969594-6d51703b-af7f-4d87-b035-1666a8bfbeec.png#averageHue=%2325282f&clientId=u25dc4089-44af-4&from=ui&id=ubabfe8e4&originHeight=571&originWidth=1446&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=99486&status=done&style=none&taskId=udd5a9141-46e4-4ca1-82b9-28454017b42&title=)<br />![屏幕截图 2024-05-22 204739.png](https://cdn.nlark.com/yuque/0/2024/png/40660095/1716382040061-b5e086ea-accf-4e86-9ce7-503a5dbe7ed8.png#averageHue=%2324282f&clientId=u25dc4089-44af-4&from=ui&id=ua15bfc25&originHeight=941&originWidth=1886&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=146198&status=done&style=none&taskId=u32a6fe9f-d2e6-4e6c-8703-389a9cacec5&title=)

1. 首先我们要注意的是visibleCount的计算，我们是对结果向上取整，**把整个列表容器填满，保证整个列表容器中不会出现多余的地方**。
2. 其次最重要的是关于对偏移量的理解，以及为什么要用偏移量。首先我们知道，在滚动的过程中列表的可视区域的列表项都是在变化的，（看一下visibleData的实现）那么在滚动的过程中，可能发生这样一个状况：滚动效果的消失，**比如说，在将第一项滚动消失后，本来是第二项在列表首位，但是 列表首位变成了第三项。**这就是因为，偏移量没有重新设置导致按照原来的偏移量（设置成scrollTop）滚动效果消失。
3. 另外，我们发现过于快速的滚动会出现白屏现象，所以我们利用rafThrottle写了一个节流函数，该函数接受一个函数，我们传入滚动函数。

```javascript
rafThrottle(fn) {
  let lock = false;
  return function (...args) {
    window.requestAnimationFrame(() => {
      if (lock) return;
      lock = true;
      fn.apply(this, args);
      lock = false;
    });
  };
}
```

<a name="AH6nm"></a>

# 不定高的虚拟函数

在我们处理一些文本的列表项时，这时列表项的高度是不确定的，我们无法像定高一样，去确定起始节点，结束节点，还有偏移量，但是我们可以使用预估列表项高度的方法先确定列表的大概形状，再调用updated的钩子函数，更新列表项的位置高度。
<a name="wkr6r"></a>

### 数据获取

与之前不同我们调用faker.js生成虚拟的文本作为列表文本。
<a name="tHrsy"></a>

### 信息获取

我们先获取新的列表数据结构<br />![屏幕截图 2024-05-22 212611.png](https://cdn.nlark.com/yuque/0/2024/png/40660095/1716384380446-6d7cc5a4-07b5-4537-9c19-631005158c44.png#averageHue=%2324282f&clientId=u25dc4089-44af-4&from=ui&id=u2f917d9a&originHeight=479&originWidth=1228&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=60967&status=done&style=none&taskId=u472af6ed-dec2-43da-8756-d0168884be1&title=)<br />再利用scrollTop和二分查找来找到起始节点<br />![屏幕截图 2024-05-22 213039.png](https://cdn.nlark.com/yuque/0/2024/png/40660095/1716384634615-965b2d13-27b2-4d96-a27b-72ea794fd3fc.png#averageHue=%23252931&clientId=u25dc4089-44af-4&from=ui&id=uacdfcd58&originHeight=260&originWidth=1312&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=44677&status=done&style=none&taskId=u36e13792-5eee-4a4a-81bf-73de5f93baf&title=)![屏幕截图 2024-05-22 213054.png](https://cdn.nlark.com/yuque/0/2024/png/40660095/1716384634623-b952152e-4417-4ba1-99cd-6aade73c6233.png#averageHue=%23252830&clientId=u25dc4089-44af-4&from=ui&id=ud037bda6&originHeight=1072&originWidth=1185&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=127691&status=done&style=none&taskId=u19feee9e-9a20-4878-92c2-2cd8368c0ad&title=)<br />在更新时，我们重新计算列表项的大小<br />![屏幕截图 2024-05-22 213054.png](https://cdn.nlark.com/yuque/0/2024/png/40660095/1716384798741-bafa24bc-3800-40d6-9aa6-cc48954ceb07.png#averageHue=%23252830&clientId=u25dc4089-44af-4&from=ui&id=u47137b13&originHeight=1072&originWidth=1185&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=127691&status=done&style=none&taskId=u09b83553-3e39-49f6-8eb6-bc919adb06c&title=)<br />获取偏移量<br />![屏幕截图 2024-05-22 213713.png](https://cdn.nlark.com/yuque/0/2024/png/40660095/1716385011553-f39e18b3-0f59-4176-baf7-fd47ae5d7a90.png#averageHue=%23252830&clientId=u25dc4089-44af-4&from=ui&id=u4b8f4a61&originHeight=646&originWidth=2167&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=122238&status=done&style=none&taskId=u1ab880a5-7b5a-4b42-a530-1be241d03c5&title=)



