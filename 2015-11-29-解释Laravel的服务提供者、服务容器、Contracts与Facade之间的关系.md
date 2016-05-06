---
layout: post
title: 解释Laravel的服务提供者、服务容器、Contracts与Facade之间的关系
excerpt: 当你接触一段时间Laravel的Service Container, Service Provider，Contracts和Facade后，也许已经知道它们是什么了，但是对于如何使用，在什么时候使用，以及它们之间的关系是什么，还不是非常清楚。 
categories: [后端开发]
tags: [Laravel,PHP]
---

# 简述

当你接触一段时间Laravel的Service Container, Service Provider，Contracts和Facade后，也许已经知道它们是什么了，但是对于如何使用，在什么时候使用，以及它们之间的关系是什么，还不是非常清楚。 而关键是如果你反复看文档，你会被它坑死，因为文档有些部分不但没有解释清楚，反而有误导的内容； 现在我们就来一次性把它们搞定；

# 基本概念

在继续本教程之前，你需要先对以上概念有基本了解，知道它们是什么；

## Service Container和 Service Provider

Service Container，也就是IOC容器的使用**并不依赖Service Provider**，例如：

``` php
$app->make('App\Models\Post');
```

这句话和 `new App\Models\Post；` 的结果完全一样； 另外你在控制器里使用构造函数，type-hint进行依赖注入，也完全和Service Provider没有半毛钱关系。

总之，你可以完全不使用Service Provider；

## Service Provider 和Contracts

如果说IOC容器的使用**并不依赖Service Provider**，那么为什么我们用composer下载扩展包的时候总是要在`config/app.php`里绑定一下Service Provider呢，有时候还需要绑定一下Facade;

理解的思路是这样的，Laravel核心类(Services)都是用接口(contracts)+实现来构成的, 如果不理解这个概念，仔细看文档接口那一章。而你在使用的时候，如果要拿到某个接口实现的实例的话，需要用到Service Container，而要用Service Container去解析一个接口，而不是直接解析一个类，这时就要用到`Service Provider`了，可以说，**Service Provider的主要功能，就是来绑定接口的**。

下我准备要讲坑爹的事情了，在讲接口绑定前，先了解一些基本的事实：

### 一些事实

``` php
$app->make('App\Models\Post');
```

你可以这样写，

``` php
$app->make('post');
```

也可以这样写，这里的post是一个别名，这个别名是造成混淆的主要地方； 这个时候你肯定在想，这样写有啥用，我去哪里关联这个别名到`App\Models\Post`呢？

### Service Provider 的 bind方法

对，就是在Service Provider里用bind方法来绑定别名：

``` php
$this->app->bind('post', function ($app) {    
	return new App\Models\Post;
});
```

这样绑定后你就可以`$app->make('post');`这样写了；然而搞个别名到目前为止也没什么卵用。没关系，稍后会讲到，它和Facade有关系；我们先来解释文档坑爹的地方：

文档是这样写这个bind方法的：

``` php
$this->app->bind('HelpSpot\API', function ($app) {    
	return new HelpSpot\API($app['HttpClient']);
});
```

哇擦，您的这第一个参数到底填的啥啊，事实上，第一个参数可以填类的全称，但是如果不是填简称，我这样绑定有任何意义么？ 后面再返回一个一样的类实例？ 咦？`$app['HttpClient']`这个是什么？？ 其实它是告诉你可以在解析类的时候可以再接着注入一个其他类的实例；文档大哥，拜托你解释一下好不好，能不能举个靠谱点的例子...

如果你到其他的扩展包中去看别人的bind的写法，你会发现千奇百怪的绑定写法，先不管他们，现在我们来看Service Provider对接口的使用方法，最最基本的原理是这样的：

``` php
//给一个接口起个别名
$this->app->bind('event_pusher', function ($app) {
	return new App\Contracts\EventPusher;
});
//指定这个接口应该解析的实例
$this->app->bind('App\Contracts\EventPusher', 'App\Services\RedisEventPusher');
```

通过这两步，我们让这个接口有了别名，也有了解析时对应的实现；

这样，我们就可以：

``` php
$app->make('event_pusher');
```

得到`App\Services\RedisEventPusher`;

## Service Provider 和 Facades

我们来看Facade的写法,比如说*`Illuminate\Support\Facades\Cache`*：

``` php
class Cache extends Facade{    
	protected static function getFacadeAccessor() { 
		return 'cache'; 
	}
}
```

这个cache就是上面提到过的别名；

下面我们来看Facade的对应关系图：

| Facade Name | Facade Class                     | Resolved Class              | Service Provider Binding Alias |
| ----------- | -------------------------------- | --------------------------- | ------------------------------ |
| Cache       | Illuminate\Support\Facades\Cache | Illuminate\Cache\Repository | cache                          |

------

所以你调用`Cache::get('user_id')`的时候，实际上是调用了**`Illuminate\Support\Facades\Cache`** 这个类，get并不是这个类的静态方法，事实上，get这个方法在Facade这个类里根本不存在，这正是它设计的本意，当get这个方法不存在的时候，它就会调用Facade基类里的**`__callStatic`**魔术方法（需要提前了解这个魔术方法），这个方法中就会把Service Provider中绑定的类（或接口）解析并返回出来，本例中也就是**`Illuminate\Cache\Repository`** 这个类，所以get其实是**`Illuminate\Cache\Repository`**这个类的方法;

然后我们在再看文档，有的Facade怎么没有别名呢？比如：

| Facade Name | Facade Class                        | Resolved Class                           | Service Provider Binding Alias |
| ----------- | ----------------------------------- | ---------------------------------------- | ------------------------------ |
| Response    | Illuminate\Support\Facades\Response | Illuminate\Contracts\Routing\ResponseFactory |                                |

------

是的，你可以直接写类的全称，而不是别名，如果你看这个`**Illuminate\Support\Facades\Response**`源码，它是这样写的：

``` php
class Response extends Facade{       
	protected static function getFacadeAccessor()    {
		return 'Illuminate\Contracts\Routing\ResponseFactory';    
	}
}
```

可以直接返回该类；

## Facade的命名空间到底是什么

我们发现，在使用**`Cache::get('user_id')`**的时候，你可以使用`use Cache`; 也可以使用`use Illuminate\Support\Facades\Cache`;

这是为什么呢？

别忘了，你在`config/app.php`里面Class Aliases 那里绑定过 Facade 别名，也就是：

``` php
 'Cache'     => Illuminate\Support\Facades\Cache::class,
```

这样绑定过，你就可以直接`use Cache`来使用Facade了；

