## 函数重载

```ts
export declare interface NavigationGuardNext {
    (): void;
    (error: Error): void;
    (location: RouteLocationRaw): void;
    (valid: boolean | undefined): void;
    (cb: NavigationGuardNextCallback): void;
}
```

这个接口 `NavigationGuardNext` 定义了多个函数重载。每个函数重载指定了不同的参数类型和返回类型。这样设计的接口可以根据传入的参数类型来选择相应的函数进行调用。

在这个接口中，可以用不同的方式调用 `NavigationGuardNext`：
- 第一个函数重载 `(): void` 表示不传入任何参数时，调用该函数将没有任何返回值。
- 第二个函数重载 `(error: Error): void` 接受一个 `Error` 类型的参数，调用该函数将没有任何返回值。
- 第三个函数重载 `(location: RouteLocationRaw): void` 接受一个 `RouteLocationRaw` 类型的参数，调用该函数将没有任何返回值。
- 第四个函数重载 `(valid: boolean | undefined): void` 接受一个布尔值或 `undefined` 类型的参数，调用该函数将没有任何返回值。
- 第五个函数重载 `(cb: NavigationGuardNextCallback): void` 接受一个 `NavigationGuardNextCallback` 类型的参数，调用该函数将没有任何返回值。

通过这样的多个函数重载，可以更灵活地使用 `NavigationGuardNext` 接口，并根据不同的参数类型选择正确的函数进行调用。

## 函数签名

```TS
export declare interface NavigationGuard {
    (to: RouteLocationNormalized, from: RouteLocationNormalized, next: NavigationGuardNext): NavigationGuardReturn | Promise<NavigationGuardReturn>;
}
```

这个接口 `NavigationGuard` 定义了一个函数签名，用于表示导航守卫（Navigation Guard）的回调函数。

该函数签名包含三个参数：
- `to: RouteLocationNormalized`：表示要导航到的目标路由位置的标准化对象。
- `from: RouteLocationNormalized`：表示当前路由位置的标准化对象。
- `next: NavigationGuardNext`：表示导航守卫完成后要调用的回调函数，用于控制导航流程。

函数可以返回以下两种类型之一：
- `NavigationGuardReturn`：表示同步地完成导航守卫并返回一个结果。
- `Promise<NavigationGuardReturn>`：表示异步地完成导航守卫并通过一个 Promise 返回结果。

导航守卫函数使用这个接口签名来定义，当路由导航触发时，它将被调用，并通过参数提供有关要导航到的目标位置、当前位置和回调函数来控制导航的行为。根据执行结果，导航守卫函数可以选择同步或异步地完成导航，并返回适当的结果或 Promise 对象。

在这个接口的定义中，使用 `{}` 包裹函数的签名是为了指定函数的类型。它表示这是一个函数类型的接口成员。

在 TypeScript 中，可以使用函数类型来定义接口成员或函数参数的类型。函数类型由函数的参数列表和返回类型组成。使用 `{}` 将函数签名包裹起来，可以明确地指定接口成员的类型为函数类型。

在像这样的接口中，使用 `{}` 是必需的，因为它指定了这是一个函数，并声明了函数的参数和返回类型。去掉 `{}` 将无法正确表示函数类型，导致语法错误。

所以，在这个接口中，使用 `{}` 包裹函数签名是必须的，不能省略。



```
declare type NavigationGuardNextCallback = (vm: ComponentPublicInstance) => any;
```

这个类型声明 `NavigationGuardNextCallback` 定义了一个函数类型，用于表示导航守卫的回调函数。

该函数类型接受一个参数 `vm`，类型为 `ComponentPublicInstance`，表示组件的公共实例。在导航守卫中，这个回调函数可以用于访问和操作组件的实例。

该函数可以返回任意类型的值 `any`。

使用这个函数类型，可以定义导航守卫中的回调函数，它接受组件实例作为参数并执行相应的操作。这样的回调函数可以在导航守卫中被调用，以实现与组件的交互和处理逻辑。

## interface

```TS
里面的[]是什么意思?
export declare interface HistoryState {
    [x: number]: HistoryStateValue;
    [x: string]: HistoryStateValue;
}
```

这个声明定义了一个接口 `HistoryState`。它描述了一个对象类型，该对象可以通过数字或字符串索引来访问，而索引对应的值必须是 `HistoryStateValue` 类型。

在 TypeScript 中，使用方括号 `[]` 表示索引签名（Index Signature），它允许你定义对象类型的索引方式。索引签名由两部分组成，一个是索引类型，另一个是索引对应的值类型。

在这个特定的接口声明中，有两个索引签名：
- `[x: number]: HistoryStateValue`：表示对象可以通过数字索引来访问，对应的值必须是 `HistoryStateValue` 类型。
- `[x: string]: HistoryStateValue`：表示对象可以通过字符串索引来访问，对应的值也必须是 `HistoryStateValue` 类型。

这样定义的 `HistoryState` 接口允许你按照数字或字符串作为索引来访问其属性值，而这些属性值都必须符合 `HistoryStateValue` 类型。

以下是一个简单示例来说明如何使用 `HistoryState` 接口：

```typescript
interface HistoryStateValue {
  // 定义 HistoryStateValue 接口的具体属性
  // ...
}

const historyState: HistoryState = {
  1: { /* 具体的 HistoryStateValue 对象 */ },
  'foo': { /* 具体的 HistoryStateValue 对象 */ },
};

const value1 = historyState[1]; // 访问数字索引为 1 的属性值
const value2 = historyState['foo']; // 访问字符串索引为 'foo' 的属性值
```

在上述示例中，`historyState` 对象符合 `HistoryState` 接口的定义。你可以通过数字索引（例如 `1`）或字符串索引（例如 `'foo'`）来访问对象的属性值，而这些属性值的类型都必须是 `HistoryStateValue`。

## Lazy

```TS
declare type Lazy<T> = () => Promise<T>;
```

`declare type Lazy<T> = () => Promise<T>;` 是一个 TypeScript 类型声明。

这个声明定义了一个泛型类型 `Lazy<T>`。`Lazy<T>` 是一个函数类型，它接受一个不带参数的函数作为参数，返回一个 Promise，该 Promise 的类型为 `T`。

具体来说，`Lazy<T>` 是一个函数类型，其签名为 `() => Promise<T>`。函数没有任何参数，函数体内部返回一个 Promise，该 Promise 的值类型为 `T`。

这个类型声明通常用于表示延迟加载（Lazy Loading）的值。通过使用 `Lazy<T>` 类型，可以定义一个函数，或者说是一个延迟加载的计算，它在需要时才会执行，并返回一个 Promise。

下面是一个示例，展示了如何使用 `Lazy<T>` 类型：

```typescript
declare type Lazy<T> = () => Promise<T>;

function fetchData(): Promise<string> {
  return new Promise((resolve) => {
    setTimeout(() => {
      resolve('Data has been fetched.');
    }, 2000);
  });
}

const lazyData: Lazy<string> = () => fetchData();

// 使用延迟加载的值
lazyData().then((data) => {
  console.log(data); // 输出: "Data has been fetched."
});
```

在上述示例中，我们定义了一个 `fetchData` 函数，它返回一个延迟加载的 Promise，该 Promise 的值类型为字符串。

然后，我们创建了一个 `lazyData` 变量，并将其类型设置为 `Lazy<string>`，即一个返回字符串类型的延迟加载函数。

我们可以通过调用 `lazyData()` 来触发延迟加载的执行，并通过 `then` 方法处理返回的 Promise 值。当 Promise 被解析时，我们可以访问到最终的数据值并进行进一步的操作。

总而言之，`declare type Lazy<T> = () => Promise<T>;` 定义了一个泛型类型，用于表示一个返回 Promise 的延迟加载函数，可以在需要时异步执行并获取结果。

## Omit排除 Omit<NavigationFailure, 'to' | 'type'>

```TS
export declare interface NavigationRedirectError extends Omit<NavigationFailure, 'to' | 'type'> {
    type: ErrorTypes.NAVIGATION_GUARD_REDIRECT;
    to: RouteLocationRaw;
}
```

`Omit<NavigationFailure, 'to' | 'type'>` 是 TypeScript 中的一个工具类型，用于从另一个类型中排除指定的属性。 在这个特定的示例中，`NavigationFailure` 是一个类型，而 `to` 和 `type` 是 `NavigationFailure` 类型中的两个属性。 `Omit<NavigationFailure, 'to' | 'type'>` 的作用是从 `NavigationFailure` 类型中排除 `'to'` 和 `'type'` 这两个属性，返回一个新类型，该新类型不包含这两个属性。 换句话说，`Omit<NavigationFailure, 'to' | 'type'>` 用于创建一个新类型，该新类型与 `NavigationFailure` 类型相同，除了不包含 `'to'` 和 `'type'` 这两个属性。 在给定的例子中，`NavigationRedirectError` 接口声明了一个继承自 `NavigationFailure` 类型（除去 `'to'` 和 `'type'` 属性）的新接口。该新接口添加了一个额外的属性 `type`，它的值是 `ErrorTypes.NAVIGATION_GUARD_REDIRECT`。



## Pick挑选出指定的属性 Pick<_PathParserOptions, 'end' | 'sensitive' | 'strict'>;

```
export declare type PathParserOptions = Pick<_PathParserOptions, 'end' | 'sensitive' | 'strict'>;
```

`Pick<_PathParserOptions, 'end' | 'sensitive' | 'strict'>` 是一个 TypeScript 中的工具类型，用于从另一个类型 `_PathParserOptions` 中挑选出指定的属性。

 具体来说，`Pick` 类型接受两个类型参数：第一个参数是要选择属性的源类型，第二个参数是要选择的属性名列表。 

在这个特定的示例中，`_PathParserOptions` 是一个类型，而 `'end'`、`'sensitive'` 和 `'strict'` 是 `_PathParserOptions` 类型中的某些属性。 

`Pick<_PathParserOptions, 'end' | 'sensitive' | 'strict'>` 的作用是从 `_PathParserOptions` 类型中挑选出包含 `'end'`、`'sensitive'` 和 `'strict'` 这几个属性的子类型，返回一个新类型。 

换句话说，`Pick<_PathParserOptions, 'end' | 'sensitive' | 'strict'>` 用于创建一个新类型，该新类型只包含来自 `_PathParserOptions` 类型的 `'end'`、`'sensitive'` 和 `'strict'` 这几个属性。 

在给定的例子中，`PathParserOptions` 是一个类型别名，它声明了一个新类型，该新类型是从 `_PathParserOptions` 类型中选择性地挑选出了 `'end'`、`'sensitive'` 和 `'strict'` 这几个属性。 



## 多接口继承

```TS
export declare interface RouteLocationPathRaw extends RouteQueryAndHash, MatcherLocationAsPath, RouteLocationOptions {
}
```

在 TypeScript 中，`extends` 关键字用于表示继承关系。在给定的代码片段中，`RouteLocationPathRaw` 接口继承了多个其他接口，使用逗号分隔它们。

 具体来说，`RouteLocationPathRaw` 接口扩展了以下接口：

1. `RouteQueryAndHash`: 这应该是另一个接口类型，它包含与路由查询参数和哈希相关的属性。
2.  `MatcherLocationAsPath`: 这也可能是另一个接口类型，与路由匹配器的位置相关，并包含某些与路径相关的属性。
3. `RouteLocationOptions`: 这是另一个接口类型，它包含路由位置的相关选项，例如 `replace`、`scrollBehavior` 等。 
4. 通过使用 `extends` 关键字和逗号分隔的方式，`RouteLocationPathRaw` 接口可以继承多个接口的属性。这使得 `RouteLocationPathRaw` 接口具有继承的接口类型的所有属性，从而扩展了其功能和用途。



## 排除 Exclude

```TS
export declare type RouteParamsRaw = Record<string, RouteParamValueRaw | Exclude<RouteParamValueRaw, null | undefined>[]>;
```

`Exclude<RouteParamValueRaw, null | undefined>[]` 的意思是从类型 `RouteParamValueRaw` 中排除了 `null` 和 `undefined`，然后作为一个数组。 

`Exclude<T, U>` 是 TypeScript 标准库中的一个类型工具，用于从类型 `T` 中排除类型 `U`。

在这种情况下，它用于从类型 `RouteParamValueRaw` 中排除了 `null` 和 `undefined`。 





```ts
export declare interface Router {
    addRoute(parentName: RouteRecordName, route: RouteRecordRaw): () => void;
}
```

这个接口 `Router` 定义了一个路由器的结构，它包含了一个方法 `addRoute`。 

`addRoute` 方法接受两个参数：`parentName` 和 `route`，分别表示父级路由名称和路由记录。

它返回一个函数，该函数用于解绑添加的路由。 

具体解释如下： - `addRoute(parentName: RouteRecordName, route: RouteRecordRaw): () => void;`：这是 `Router` 接口中的方法签名。    

- `parentName: RouteRecordName`：这是一个参数，类型为 `RouteRecordName`，表示父级路由的名称。`RouteRecordName` 应该是一个字符串类型，用于标识父级路由。    
- - `route: RouteRecordRaw`：这也是一个参数，类型为 `RouteRecordRaw`，表示要添加的路由记录。`RouteRecordRaw` 是一种描述路由配置的接口或类型。    
- - `(): void`：这是方法的返回类型注解，表示这个方法没有返回值。    
-  返回的函数：此方法返回一个函数，该函数用于解绑添加的路由。这个函数不接受任何参数，也没有返回值。 这个接口定义了一个 `Router` 类型，用于描述具有 `addRoute` 方法的路由器对象。使用该接口的目的是确保在使用路由器实例时，我们可以调用 `addRoute` 方法来添加路由，并且可以通过返回的函数进行解绑操作。





```ts
export declare const RouterLink: _RouterLinkI;

/**
 * Typed version of the `RouterLink` component. Its generic defaults to the typed router, so it can be inferred
 * automatically for JSX.
 *
 * @internal
 */
export declare interface _RouterLinkI {
    new (): {
        $props: AllowedComponentProps & ComponentCustomProps & VNodeProps & RouterLinkProps;
        $slots: {
            default?: ({ route, href, isActive, isExactActive, navigate, }: UnwrapRef<ReturnType<typeof useLink>>) => VNode[];
        };
    };
    /**
     * Access to `useLink()` without depending on using vue-router
     *
     * @internal
     */
    useLink: typeof useLink;
}

```

- `export declare const RouterLink: _RouterLinkI;` 这行代码表示将名为 `RouterLink` 的常量导出，并将其类型声明为 `_RouterLinkI`。这意味着可以在其他文件中使用 `RouterLink` 变量，并在编译时进行类型检查。 

- `export declare interface _RouterLinkI { ... }` 这个接口 `_RouterLinkI` 定义了一个类型，描述了一个 `_RouterLinkI` 类型的对象的结构。该接口有两个成员：
    - `new (): { ... }`：这是一个构造函数签名，表示 `_RouterLinkI` 的实例应该具有这个结构。构造函数返回一个对象，该对象具有以下成员：  
        -  `$props`：一个包含了各种属性的对象，其中包括 `AllowedComponentProps`、`ComponentCustomProps`、`VNodeProps` 和 `RouterLinkProps` 的属性。  
        -  `$slots`：一个包含默认插槽的对象，可以在渲染时访问该插槽。 
    -  `useLink: typeof useLink`：描述了一个静态成员。它指定了一个叫做 `useLink` 的函数的类型，该函数与 `typeof useLink` 相同。
        - 根据注释指示，该函数可能是用于与 `vue-router` 不依赖的情况下访问路由链接的函数。 这段代码的目的是定义 `_RouterLinkI` 接口，以描述一个具有特定结构和成员的对象类型，用于实现类型安全的组件编写和处理路由链接。`RouterLink` 常量被声明为该接口类型，以便在其他文件中使用该常量，并引用该类型的结构和成员。





```ts
(() => Promise<{ default: RouteComponent }>)
```

`(() => Promise<{ default: RouteComponent }>)` 是一个函数类型的定义，表示一个返回 Promise 对象的函数。 具体解释如下： 

-  `() => ...` 是一个箭头函数的定义，它表示这是一个没有参数的函数。 

-  `Promise<{ default: RouteComponent }>` 表示返回一个 Promise 对象，该对象的解析值是一个包含 `default` 属性且类型为 `RouteComponent` 的对象。 这种函数类型的定义通常在代码中用于表示动态加载的组件。

- 在这段代码中，对应的路由组件被封装在一个 Promise 中，并且通过 `default` 属性访问该组件。这种方式可以在需要时延迟加载组件，并在加载完成后才使用它，以实现按需加载和提升性能的目的。 在实际使用中，可以通过调用这个函数来获取对应的 Promise，并使用 `then` 方法或 `async/await` 语法来处理 Promise 的结果，以获得实际的路由组件。

- ```TS
    const routeComponentPromise = () => import('./about/index.vue');
    
    routeComponentPromise().then((module) => {
      const routeComponent = module.default;
      // 处理路由组件
    });
    
    // 或者使用 async/await
    async function loadRouteComponent() {
      const module = await routeComponentPromise();
      const routeComponent = module.default;
      // 处理路由组件
    }
    ```

-  在以上示例中，`routeComponentPromise` 是一个返回 Promise 的函数。通过调用该函数，我们可以获取到一个包含路由组件的模块对象。然后通过访问 `default` 属性，我们可以获得实际的路由组件，进行后续的处理和使用。



















.
