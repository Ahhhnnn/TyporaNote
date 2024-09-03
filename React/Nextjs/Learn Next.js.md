# Learn Next.js

Learn Next.js:https://nextjs.org/learn/dashboard-app#what-well-be-building



# Demo

创建一个nextjs项目

注意需要使用pnpm

```shell
npm install -g pnpm
```

```shell
npx create-next-app@latest nextjs-dashboard --example "https://github.com/vercel/next-learn/tree/main/dashboard/starter-example" --use-pnpm
```

![image-20240901003924978](assets/image-20240901003924978.png)



# 创建布局和页面

简单理解：

layout.tsx:固定的文件名，用于创建布局，与该文件一个目录下的路由文件夹都会自动归属到该布局下

page.tsx:固定的文件名，next只会渲染page页面



在app目录下的文件夹名称 默认就是路由的名称，可以在文件夹内在创建page.tsx 就是一个新的页面(使用文件夹创建新的路由段)



![image-20240901215845464](assets/image-20240901215845464.png)

![image-20240901215924700](assets/image-20240901215924700.png)

添加布局，用于多个页面之间的UI共享

![image-20240901220949499](assets/image-20240901220949499.png)

![image-20240901221008169](assets/image-20240901221008169.png)



##  在页面之间导航

正常情况下使用导航 会使用<a>标签 ，但是会用a标签会发现整个页面都会进行刷新

在Next中提供了next/Link 标签进行页面间的导航



![image-20240901222532119](assets/image-20240901222532119.png)

通过**clsx** 动态添加css类型

```js
className={clsx(
    'flex h-[48px] grow items-center justify-center gap-2 rounded-md bg-gray-50 p-3 text-sm font-medium hover:bg-sky-100 hover:text-blue-600 md:flex-none md:justify-start md:p-2 md:px-3',
    {
        'bg-sky-100 text-blue-600': pathname === link.href,
    },
)}
```



# 流媒体传输

**第一种：**

直接在需要流传输的页面同级别目录下创建一个loading.tsx

```js
import DashboardSkeleton from '@/app/ui/skeletons'

export default function Loading() {
    return <DashboardSkeleton />;
}
```

该页面会先行直接返回到浏览器，知道需要显示的页面加载完毕。

（loading.tsx是一个基于 Suspense 构建的特殊 Next.js 文件，它允许您创建后备 UI 以在页面内容加载时显示为替换。）

可以在哎loading.tsx 中添加不同的组件用来渲染加载骨架





**第二种：**
使用<Suspense>组件

```js
import { Suspense } from 'react';
```

组件用法：

然后，从 React 导入`<Suspense>` ，并将其包装在`<RevenueChart />`周围。您可以向其传递一个名为`<RevenueChartSkeleton>`的后备组件。

```js
<Suspense fallback={<RevenueChartSkeleton/>}>
   <RevenueChart/>
</Suspense>
```



# 条件查询

使用URL搜索参数的好处：

- **可添加书签和可共享的 URL** ：由于搜索参数位于 URL 中，因此用户可以为应用程序的当前状态（包括其搜索查询和过滤器）添加书签，以供将来参考或共享
- **服务器端渲染和初始加载**：可以在服务器上直接使用URL参数来渲染初始状态，从而更容易处理服务器渲染。
- **分析和跟踪**：直接在 URL 中进行搜索查询和过滤器可以更轻松地跟踪用户行为，而无需额外的客户端逻辑。



NextJS提供的几个钩子函数：

**`useSearchParams`** - 允许您访问当前 URL 的参数。例如，此 URL 的搜索参数 `/dashboard/invoices?page=1&query=pending` 看起来像这样： `{page: '1', query: 'pending'}` 

**`usePathname`** - 让您读取当前 URL 的路径名。例如，对于路由`/dashboard/invoices` ， `usePathname`将返回`'/dashboard/invoices'` 

**`useRouter`** - 以编程方式启用客户端组件内的路由之间的导航。您可以使用[多种方法](https://nextjs.org/docs/app/api-reference/functions/use-router#userouter)。



**实现条件查询的步骤：**

1、捕获用户的输入。

2、使用搜索参数更新 URL。（前面讲了使用URL参数的好处，也可以直接获取参数进行查询）

3、保持 URL 与输入字段同步。

4、更新表以反映搜索查询。



- `"use client"` - 这是一个客户端组件，这意味着您可以使用事件侦听器和挂钩。

![image-20240903231226133](assets/image-20240903231226133.png)

```js
'use client';

import { MagnifyingGlassIcon } from '@heroicons/react/24/outline';

export default function Search({ placeholder }: { placeholder: string }) {
  return (
    <div className="relative flex flex-1 flex-shrink-0">
      <label htmlFor="search" className="sr-only">
        Search
      </label>
      <input
        className="peer block w-full rounded-md border border-gray-200 py-[9px] pl-10 text-sm outline-2 placeholder:text-gray-500"
        placeholder={placeholder}
      />
      <MagnifyingGlassIcon className="absolute left-3 top-1/2 h-[18px] w-[18px] -translate-y-1/2 text-gray-500 peer-focus:text-gray-900" />
    </div>
  );
}

```

钩子函数的使用方法

如果调用方法后返回的是一个常量则直接定义

**如果返回的是一个方法 则需要使用{}框起来**

 比如`const {replace} = useRouter();`

```js
import {useSearchParams, usePathname, useRouter} from "next/navigation";
const params = new URLSearchParams(searchParams);
const pathname = usePathname();
const {replace} = useRouter();
```

使用如下方法进行导航

```js
replace(`${pathname}?${params.toString()}`);
```

该函数执行以下操作：
将pathname与参数查询字符串组合：${pathname}?${params.toString()}。
使用replace方法导航至新的路径。此函数用于客户端路由跳转。



**注意**在：

钩子函数只能在Hooks必须在函数组件的顶层调用，不能在任何其他位置调用。

![image-20240903233229584](assets/image-20240903233229584.png)



Page页面默认可以接受的参数：

params：接受URL路径的参数

![image-20240903233954720](assets/image-20240903233954720.png)

searchParams：接受URL参数

![image-20240903234010271](assets/image-20240903234010271.png)



## 去抖

如果每次Onchange都会监听并查询 那么可能会导致大量的查询，此时需要进行去抖

可以自行通过计时器实现，也可以引入第三方包实现

使用debounce 实现去抖

```js
pnpm i use-debounce
```

```js
// ...
import { useDebouncedCallback } from 'use-debounce';
 
// Inside the Search Component...
const handleSearch = useDebouncedCallback((term) => {
  console.log(`Searching... ${term}`);
 
  const params = new URLSearchParams(searchParams);
  if (term) {
    params.set('query', term);
  } else {
    params.delete('query');
  }
  replace(`${pathname}?${params.toString()}`);
}, 300);
```

300 ms内用户没有键停，那么不会发起请求





# 分页查询

查看分页组件： [pagination.tsx](../../../../code/react/nextjs-dashboard/app/ui/invoices/pagination.tsx) 



