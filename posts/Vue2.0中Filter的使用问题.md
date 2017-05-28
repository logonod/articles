> vue2.0里，不再有自带的过滤器，需要自己定义过滤器

在 Vue1.0 中内置了几种实用的过滤器函数如 `uppercase` ，但在 Vue2.0 中这些方法都被废除了需要自己定义过滤器。

定义的方法：注册一个自定义过滤器，它接收两个参数：过滤器 ID 和 过滤器函数，其中过滤器函数接收多个参数。

举个栗子：

```javascript
//main.js
Vue.filter('reverseStr', function(value) {
    return value.split('').reverse().join('')
});

//*.vue
<template>
	<div>{{ content | reverseStr }}</div>
</template>

<script>
export default {
  name: 'component-name',
  data () {
    return {
      content: 'abcd'
    }
  }
}
</script>

//render result
<div>dcba</div>
```

看到这里熟悉 Shell 管道命令的同学就会感觉很熟悉，没错 Vue 的过滤器操作符 `|` 和 Shell 命令一样，能将上一个过滤器输出内容作为下一个过滤器的输入内容，也就是说 Vue 允许你这样使用过滤器：

```javascript
//main.js
Vue.filter('removeNum', function (value) {
  return value.replace(/[^0-9]/g, '');
})

//*.vue
<template>
	<div>{{ content | reverseStr | removeNum }}</div>
</template>

<script>
export default {
  name: 'component-name',
  data () {
    return {
      content: 'abcd1234'
    }
  }
}
</script>

//render result
<div>dcba</div>
```

是不是很好很强大？！但在 Vue2.0 中使用过滤器我遇到一个这样的场景，我需要在 `v-html` 指令中使用过滤器，如下：
```javascript
//*.vue
<template>
	<div class="markdown" :v-html="content | marked"></div>
</template>

<script>
export default {
  name: 'component-name',
  data () {
    return {
      content: '## 标题**哈哈**'
    }
  }
}
</script>
```

这种写法在 Vue1.0 中并没有问题，但是在 Vue2.0 中抛出错误：

> property or method "marked" is not defined on the instance but referenced during render. Make sure to declare reactive data properties in the data option

最终查阅官方文档给出的解释是：

> Filters can now only be used inside text interpolations ({{}} tags). In the past we've found using filters with directives such as v-model, v-on etc. led to more complexity than convenience, and for list filtering on v-for it is more appropriate to move that logic into JavaScript as computed properties.

也就是说过滤器现在只能用在两个地方：**mustache 插值和** `v-bind` **表达式**。在 `v-model` 、`v-on` 、`v-for` 等指令时 Vue 还是推荐我们将该逻辑通过 `computed`  来计算属性。所以我们需要进行改写：

```javascript
<template>
	<div class="markdown">{{ markedContent }}</div>
</template>

<script>
export default {
  name: 'component-name',
  data () {
    return {
      content: '## 标题**哈哈**'
    }
  },
  computed: {
    markedContent () {
      return marked(content)
    }
  }
}
</script>
```

