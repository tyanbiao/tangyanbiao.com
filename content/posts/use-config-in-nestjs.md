---
title: NestJS 配置维护
date: 2022-10-19T19:45:00+08:00
---



NestJS 提供了开箱即用的模块 `@nestjs/config` 用来管理 NestJS 应用的配置，[官方文档](https://docs.nestjs.com/techniques/configuration#configuration) 已经提供了足够详细的案例，很容易就能上手使用，不过在实际开发过程中还是会遇到一些没有被很好解决的痛点，于是写一篇文章总结一下。

## 在 `main.ts` 中使用

```typescript
const configService = app.get<ConfigService>(ConfigService)   
await app.listen(configService.get('SERVER_PORT'))
```

## 在 `NestFactory.create()` 中使用

由于 `ConfigService`是由 Nest 管理的，在调用 `NestFactory.create` 之后 `ConfigServic` 才会被实例化，此时想要获取配置就得另外想办法。我的解决办法是通过 `NestFactory.createApplicationContext()`创建一个没有实际意义的 configApp，通过 configApp 获取 `ConfigService`，然后再创建真正的 app。

第一步先把 `ConfigModule` 抽离出来，configApp 只需要引入这一个模块。

```typescript
// custom-config.module.ts
@Module({
    imports: [
        ConfigModule.forRoot({
            isGlobal: true,
            envFilePath: ['.env.development'],
            ignoreEnvFile: process.env.NODE_ENV === 'production',
            load: [redisConfig],
            validationOptions: {
                abortEarly: true,
            },
        }),
    ],
})
export class CustomConfigModule {}

```

```typescript
// app.module.ts
@Module({
    imports: [CustomConfigModule, TaskModule, QueueModule],
})
export class AppModule {}
```

单独为 `CustomConfigModule` 创建 configApp 

```typescript
// main.ts
const configApp = await NestFactory.createApplicationContext(
    CustomConfigModule
)
const config = configApp.get<ConfigService>(ConfigService)
const options = {
    host: config.get('redis.host'),
    port: config.get('redis.port'),
}
await configApp.close()
const app = await NestFactory.createMicroservice<MicroserviceOptions>(
    AppModule,
    {
        transport: Transport.REDIS,
        options,
    }
)
```

## 在无法注入依赖地方使用

这也是一个常见的需求，`ConfigService` 只能在能注入依赖的地方使用，不过 `ConfigService` 实例化完成后，在配置文件中定义的配置会被加载到`process.env`中，此时可以通过`process.env`获取配置。

```typescript
// redis.config.ts
export default registerAs('redis', () => {
    const schema = Joi.object<{
        host: string
        port: number
    }>({
        port: Joi.number().required(),
        host: Joi.string().required(),
    })

    const { error, value } = schema.validate({
        port: process.env.REDIS_PORT,
        host: process.env.REDIS_HOST,
    })

    if (error) {
        throw error
    }
    return value
})
```

```typescript
// app.module.ts
@Module({
    imports: [
        ConfigModule.forRoot({
            envFilePath: ['.env.development'],
            load: [redisConfig],
        }),
    ],
})
export class AppModule {}
```

然后在需要使用配置的地方直接加载就行了，需要注意的是必须在 app 启动后才能获取到正确的值。

```typescript
// util.ts
import redisConfig from './redis.config.ts'

const options = redisConfig() // 此时app尚未启动完成，获取到的值不正确

export function getConfig() {
    const options = redisConfig() // 正确的获取方式
}

```

## 配合 Prisma

Prisma 有自己的一套配置，默认使用 `.env` 作为配置文件，但是实际使用中通常是需要在不同的环境加载不同的配置文件，要想为 Prisma 加载不同的环境变量，需要通过 `dotenv-cli` 来实现。

安装 `dotenv-cli`

```shell
pnpm add -D dotenv-cli
```

在 `package.json`中配置一下 `scripts`

```json
{
	"prisma:dev": "dotenv -e .env.development prisma",
    "prisma:prod": "dotenv -e .env.production prisma"
}
```

执行 `pnpm prisma:dev`会加载 `.env.development`，执行 `pnpm prisma:prod`则会加载`.env.production`



