# 数组更新检测

Vue将被侦听的数组的变更方法进行了包裹，所以它们也会触发视图更新：

- push()
- pop()
- unshift()
- shift()
- splice()
- sort()
- reverse()

上面这些方法会直接修改原数组，但是某些方法会生成新的数组而不改变原数组：

- filter()
- slice()
- concat()

# v-model中的修饰符

## `.lazy`修饰符

默认情况下，v-model再进行双向绑定时绑定的是`input`事件，因此输出的内容和绑定的内容会一直保持同步

当使用`v-model.lazy="xxx"`时，会将绑定的事件变为`change`事件，只有在提交时才会触发（例如 回车）

## `.number`修饰符

默认情况下v-model绑定的值在输入的时候会变为`str`类型

当使用`v-model.number="xxx"`时，会将输入的内容从最开始的连续数字部分截取转化为`number`类型

## `.trim`修饰符

可以删除输入值前后的空格