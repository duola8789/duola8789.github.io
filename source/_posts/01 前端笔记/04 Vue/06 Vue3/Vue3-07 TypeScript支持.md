---
title: Vue3-07 TypeScript支持
top: false
date: 2021-05-19 17:27:00
updated: 2021-05-19 17:27:02
tags:
- Vue
- Vue3
categories: Vue
---

Vue3学习笔记-07 TypeScript支持

<!-- more -->

Vue3对TypeScript的支持比Vue2更加强大，因为Vue3本身就是用TypeScript编写的

# 项目创建

可以使用Vue CLI来直接生成使用TypeScript的新项目

```BASH
# 1. Install Vue CLI, 如果尚未安装
npm install --global @vue/cli@next

# 2. 创建一个新项目, 选择 "Manually select features" 选项
vue create my-project-name

# 3. 如果已经有一个不存在TypeScript的 Vue CLI项目，请添加适当的 Vue CLI插件：
vue add typescript
```

组件的`<srcipt>`的`lang`属性应该设置为`ts`

# 定义组件

让TypeScript正确推断Vue组件选项中的类型，需要使用`defineComponent`全局方法定义组件

```JS
import { defineComponent } from 'vue'

export default defineComponent({
  // 已启用类型推断
})
```

# 选项式API的类型推断

通过`defineComponent`后，定义在`data`中的数据回自动被推断类型

```JS
const Component = defineComponent({
  data() {
    return {
      count: 0
    }
  },
  mounted() {
    const result = this.count.split('') // => Property 'split' does not exist on type 'number'
  }
})
```

这个时候，`count`会被推断为数字类型，但是如果是一个复杂的类型或者结果，需要使用断言来对进行强制转换

```JS
interface Book {
  title: string
  author: string
  year: number
}

const Component = defineComponent({
  data() {
    return {
      book: {
        title: 'Vue 3 Guide',
        author: 'Vue Team',
        year: 2020
      } as Book
    }
  }
})
```

# 计算属性的类型

计算属性类型需要显式的声明计算属性的返回类型


```JS
import { defineComponent } from 'vue'

const Component = defineComponent({
  data() {
    return {
      message: 'Hello!'
    }
  },
  computed: {
    // 需要注解
    greeting(): string {
      return this.message + '!'
    }

    // 在使用 setter 进行计算时，需要对 getter 进行注解
    greetingUppercased: {
      get(): string {
        return this.greeting.toUpperCase();
      },
      set(newValue: string) {
        this.message = newValue.toUpperCase();
      },
    },
  }
})
```

# Props的类型

Vue对定义了`type`的Prop执行运行时验证，需要通过`PropType`强制转换构造函数，将类型提供给TypeScript

```JS
import { defineComponent, PropType } from 'vue'

interface Book {
  title: string
  author: string
  year: number
}

const Component = defineComponent({
  props: {
    name: String,
    success: { type: String },
    callback: {
      type: Function as PropType<() => void>
    },
    book: {
      type: Object as PropType<Book>,
      required: true
    }
  }
})
```

涉及到Prop的`validators`和`default`进行类型推断时，要注意`this`的类型，需要使用箭头函数，或者明确提供`this`参数

```JS
import { defineComponent, PropType } from 'vue'

interface Book {
  title: string
  year?: number
}

const Component = defineComponent({
  props: {
    bookA: {
      type: Object as PropType<Book>,
      // 请务必使用箭头函数
      default: () => ({
        title: 'Arrow Function Expression'
      }),
      validator: (book: Book) => !!book.title
    },
    bookB: {
      type: Object as PropType<Book>,
      // 或者提供一个明确的 this 参数
      default(this: void) {
        return {
          title: 'Function Expression'
        }
      },
      validator(this: void, book: Book) {
        return !!book.title
      }
    }
  }
})
```

# Emit类型

Vue3的选项中新增了`emits`选项，它的主要目的就是避免的Vue2.x中隐式的定义`emit`事件的弊端，并且可以显示的指明Emit的载荷类型

不需要类型注解时，`emits`接受一个数组，数组成员是字符串组成的事件名，需要类型注解时，接受的参数是这个Emit事件的校验函数，这样就可以校验每一个Emit的参数和返回值（与Prop的Validator类似）

```JS
const Component = defineComponent({
  emits: {
    addBook(payload: { bookName: string }) {
      // perform runtime 验证
      return payload.bookName.length > 0
    }
  },
  methods: {
    onSubmit() {
      this.$emit('addBook', {
        bookName: 123 // 类型错误！
      })
      this.$emit('non-declared-event') // 类型错误！
    }
  }
})
```

# 与组合式API一起使用

在`setup()`函数中，不需要为`props`参数传递类型，它会从组件的Prop选项中推断类型

# `ref`类型

Refs类型会根据初始值进行推断，如果Ref是一个复杂类型，可以通过传给`ref`一个泛型类指定类型

```JS
const year = ref<string | number>('2020') // year's type: Ref<string | number>

year.value = 2020 // ok!
```

# `reactive`类型

声明`reactive`类型可以使用接口，作为泛型传给`reative`，或者作为变量的类型也可以

```JS
import { defineComponent, reactive } from 'vue'

interface Book {
  title: string
  year?: number
}

export default defineComponent({
  name: 'HelloWorld',
  setup() {
    const book = reactive<Book>({ title: 'Vue 3 Guide' })
    // or
    const book: Book = reactive({ title: 'Vue 3 Guide' })
    // or
    const book = reactive({ title: 'Vue 3 Guide' }) as Book
  }
})
```
