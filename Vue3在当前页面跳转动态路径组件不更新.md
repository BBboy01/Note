# 问题：

> 例如当前路径为`/note/1`，并且处于当前页面，点击对应的子组件元素要跳转的路径为`/note/2`时，浏览器URL变为`/note/2`但是父组件页面内容却没有响应式的根据URL最后的数字发生改变

# 解决方法：

> 在路由守卫`onBeforeRouteUpdate`中修改`id`，并在生命周期`onUpdated`中再次实现需要更新内容的逻辑

子组件：

```js
setup() {
	const goInfo = id => {
      router.push({ path: `/note/${id}` })
	}
    
    return { goInfo }
}
```

父组件：

```js
import { ref, onMounted, onUpdated } from 'vue'
import { useRoute, onBeforeRouteUpdate } from 'vue-router'

setup () {
    const route = useRoute()

    let id = ref(route.path.split('/').slice(-1)[0])

    let mdHtml = ref('')

    onMounted(async () => {
      mdHtml.value = (await getSingleNote(id.value)).content
    })

    onUpdated(async () => {
      mdHtml.value = (await getSingleNote(id.value)).content
    })

    onBeforeRouteUpdate(to => {
      id.value = to.path.split('/').slice(-1)[0]
    })

    return {
      id,
      mdHtml,
    }
  }
```

