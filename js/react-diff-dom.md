## react-diff算法设计思路

#### Diff算法 - 根元素类型不同

- 如果两棵树的根元素类型不同，React会销毁旧树，创建新树

```
// 旧树
<div>
  <Counter />
</div>

// 新树
<span>
  <Counter />
</span>

执行过程：destory Counter -> insert Counter
```

#### Diff算法 - 同类型不同属性

- 对于类型相同的React DOM 元素，React会对比两者的属性是否相同，只更新不同的属性
- 当处理完这个DOM节点，React就会递归处理子节点。

```
// 旧
<div className="before" title="stuff" />
// 新
<div className="after" title="stuff" />
只更新：className 属性

// 旧
<div style={{color: 'red', fontWeight: 'bold'}} />
// 新
<div style={{color: 'green', fontWeight: 'bold'}} />
只更新：color属性
```

#### Diff算法 - 增删子节点

- 1 当在子节点的后面添加一个节点，这时候两棵树的转化工作执行的很好

```
// 旧
<ul>
  <li>first</li>
  <li>second</li>
</ul>

// 新
<ul>
  <li>first</li>
  <li>second</li>
  <li>third</li>
</ul>

执行过程：
React会匹配新旧两个<li>first</li>，匹配两个<li>second</li>，然后添加 <li>third</li> tree
```

- 2 但是如果你在开始位置插入一个元素，那么问题就来了：

```
// 旧
<ul>
  <li>Duke</li>
  <li>Villanova</li>
</ul>

// 新
<ul>
  <li>Connecticut</li>
  <li>Duke</li>
  <li>Villanova</li>
</ul>

在没有key属性时执行过程：
React将改变每一个子删除重新创建，而非保持 <li>Duke</li> 和 <li>Villanova</li> 不变
```

#### Diff算法 - key 属性

> 为了解决以上问题，React提供了一个 key 属性。当子节点带有key属性，React会通过key来匹配原始树和后来的树。



```
// 旧
<ul>
  <li key="2015">Duke</li>
  <li key="2016">Villanova</li>
</ul>

// 新
<ul>
  <li key="2014">Connecticut</li>
  <li key="2015">Duke</li>
  <li key="2016">Villanova</li>
</ul>
执行过程：
现在 React 知道带有key '2014' 的元素是新的，对于 '2015' 和 '2016' 仅仅移动位置即可 
```

- 说明：key属性在React内部使用，但不会传递给你的组件
- 推荐：在遍历数据时，推荐在组件中使用 key 属性：`<li key={item.id}>{item.name}</li>`
- 注意：**key只需要保持与他的兄弟节点唯一即可，不需要全局唯一**
- 注意：**尽可能的减少数组index作为key，数组中插入元素的等操作时，会使得效率底下**