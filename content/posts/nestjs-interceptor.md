---
title: "NestJS 拦截器"
date: 2022-09-21T16:00:00+08:00
draft: false
---

## 拦截器（interceptor）
![](https://images-1252366546.cos.ap-guangzhou.myqcloud.com/notes/2022-9-21/Interceptors_1.png)
官方文档介绍：
>拦截器是一个实现了 实现 `NestInterceptor` 接口并使用 `@Injectable()` 装饰器注解的类。

拦截器主要有这几个作用
- 在路由方法执行的前后处理额外的逻辑
- 转换从路由方法返回的结果
- 转换从路由方法抛出的异常
- 扩展路由方法的行为
- 完全重写路由函数

## 修改 Response
借助拦截器，可以轻松修改 Response，比如把 `bigint` 类型转换为 `string` ，去掉 `user` 的 `password` 属性，将自增的 `id` 重新编码等。同样的，也可以自定义一个 `RestfulInterceptor` 轻松地将 Response 包装为统一的格式。

```ts
export interface RestfullResponse<T> {
    data: T
    code: number
    message: string
}


@Injectable()

export class RestfulInterceptor<T> implements NestInterceptor<T, RestfullResponse<T>> {
    intercept(
        context: ExecutionContext,
        next: CallHandler
    ): Observable<RestfullResponse<T>> {
        return next
            .handle()
            .pipe(map((data) => ({ data, code: 0, message: 'success' })))
    }
}
```

## 忽略拦截器
日常开发中，通常会把拦截器绑定到全局或者绑定到 `Controller` 上，这样不必为每一个路由方法都绑定一次拦截器，但是很多时候又想给单独在一个路由上取消绑定拦截器。幸运的是，NestJS 可以为路由自定义 `metadata` ，我们可以在拦截器中获取自定义的 `metadata` 来执行特定的逻辑。
新建一个 `ignore-interceptor.decorator.ts` 文件。

```TypeScript
import { SetMetadata } from '@nestjs/common'

export const IGNORED_INTERCEPTORS = 'IGNORED_INTERCEPTORS'

export const IgnoreInterceptors = (...interceptors: string[]) => 
    SetMetadata(IGNORED_INTERCEPTORS, interceptors)
```

然后在 `Controller` 中引入 `IgnoreInterceptors`  装饰器。

```ts
@Get()
@IgnoreInterceptors(HashidsInterceptor.name, RestfulInterceptor.name)
list() {
 //...
}
```

在上面的 `RestfulInterceptor` 中继续修改，增加一个 `shouldIgnore` 方法，获取路由的 `metadata`，检查当前拦截器是否在 `IGNORED_INTERCEPTORS` 中，如果在，就不做任何额外处理。

```ts
@Injectable()
export class RestfulInterceptor<T> implements NestInterceptor<T, RestfullResponse<T>> {
    constructor(readonly reflector: Reflector) {}

    intercept(
        context: ExecutionContext,
        next: CallHandler
    ): Observable<RestfullResponse<T>> {
        if (this.shouldIgnore(context, RestfulInterceptor.name)) {
            return next.handle()
        }
        return next
            .handle()
            .pipe(map((data) => ({ data, code: 0, message: 'success' })))
    }

	shouldIgnore(context: ExecutionContext, name: string): boolean {
        const interceptors = this.reflector.getAllAndOverride<string[]>(
            IGNORED_INTERCEPTORS,
            [context.getHandler(), context.getClass()]
        )
        if (interceptors && interceptors.length > 0) {
            return interceptors.includes(name)
        }
        return false
    }
}

```
