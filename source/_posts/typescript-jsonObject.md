---
title: json2typescritp 使用总结
date: 2017/12/4
categories: typescript
tag: 	
	- typescript
---

# 为什么需要json2typescript

在前端项目中会存在需要将json格式的字符串进行序列化，或将对象反序列化为json格式的字符串。虽然我们可以通过`JSON.stringify()`和`JSON.parse()`两个来完成序列和反序列化，但是`typescript`中我们会往往会将Object声明成一个类。首先我们先看一下在angular中从后端获取到数据后的例子。

```typescript
interface ItemsResponse {
  results: string[];
}
```

当我们发起 `HttpClient.get` 调用时，传入一个类型参数：

``` typescript
// 注意这里在尖括号内表示的返回结构的类型
this.http.get<ItemsResponse>('/api/items')
 .subscribe(data => {
  this.results = data['results'];
});
```

在这个官方文档的例子里，我们可以了解到angular默认已经帮我们做掉了`JSON.parse()`序列化的操作，在拿到data的时候就已经是`Object`了，并且我们可以通过`this.http.get<DataStruct>`的方式，在尖括号里规定返回的数据的结构类型。但是在实践中，我们有时需要的不仅仅是规定返回数据的结构类型（interface），而是需要直接将返回的数据实例化为一个对象。例如：

```typescript
export class Country {
   
  name: string;
    
  cities: City[];
    
  // 统计国家的城市数量
  countCity(): number {
    return this.cities.length;
  }
 
}
```

从后端获取了`Country`类型的数据，并且进行了隐似转型。

``` typescript
// 注意这里在尖括号内表示的返回结构的类型
this.http.get<Country>('/api/items')
 .subscribe(data => {
  console.log(data instanceof Country); // true
  console.log(data); // {name: "China", cities: ["Beijing", "Shanghai"]}
  data.countCity(); // error
});
```

从上面我们可以看到，当我们拿到后端返回的数据后，因为我们在尖括号内加入了类型，所以angular帮我们做了隐似转型的工作，所以 `data instan  ceof Country`返回的是`true`，但是当我们将`data`直接打印出来的时候，却看到其仍然只是简单的`jsonObject`都，即他并不是一个真正的`Country`实例，比如在他的原型上都没有`countCity()`这个方法。为了能将从后端拿到的数据转变了一个类的实例，我们便需要通过`new Country()`来显示的实例化，但这并不是我们想要的结果，我们想要的可以从后端拿到数据时可以直接将其序列化成一个对象的实例。

这个时候我们就需要用到`json2typescript`来完成序列化的工作。当然除了序列化，在反序列化时我们也需要做一些特殊的处理。例如将前端使用的在反序列化时变为另一个字段给后端使用等，这也可以使用该插件来帮助我们完成，而不需要现实的写数据处理的逻辑来污染业务代码。



# 安装和配置

其实使用`json2typescript`主要分两步，第一步是在我们的项目中引入`json2typescript`。

首先我们需要进入到我们的项目文件夹中，然后在命令行中输入：

```shell
npm install json2typescript
```

然后等待安装完成。然后进入我们项目中，打开`tsconfig.json`文件。如下图在**compilerOptions**中加入如下配置：

```json
{
  "compilerOptions": {
    [...]
    "experimentalDecorators": true, 
    "emitDecoratorMetadata": true,
    [...]
}
```

这是项目中加入配置行后的截图：

![tsconfig.json配置](/img/17-12/18-1.png)

这两行配置的作用便是在你的项目中启用`json2typescript`。之后我们便可以开始使用这个插件了。



# 使用方法

json2typescript的使用主要是两个地方，一个是在获取到数据是需要先调用实例化一个`JsonConvert`的实例然后通过调用它提供的方法进行序列化和反序列化，第二个地方则是对所映射的对象的进行配置。

## 在映射的类上进行配置

首先我们需要在我们需要序列化的类中加入`json2typescript`为我们提供的装饰器来装饰类

```typescript
// 注意是从json2typescript包中引入的
import {JsonObject} from "json2typescript"; 

@JsonObject
export class User {}
```

然后使用`@JsonProperty`来装饰属性

```typescript
import {JsonObject, JsonProperty} from "json2typescript"; 

@JsonObject
export class User {
    @JsonProperty("jsonPropertyName", String, false)
    name: string = undefined;
}
```

> 注意：被装饰的属性必须有初始化的值或者初始化为undefined，否则会出现异常。

我们可以看到`@JsonProperty`有三个属性，这三个属性分别表示序列化时的属性名、映射关系和是否在json中存在。

### 第一个参数

第一个参数其实就是在 `jsonObject`中这个字段所对应的字段名，主要原因是往往后端所给我们的属性名并不是前端真正想使用的名字，所以我们可以在这里给他定义一个新的字段名。

> The first parameter of `@JsonProperty` is the JSON object property name. It happens that the property names given by the server are very ugly. Here you can map any json property name to the `User` property `name`. In our case, `json["jsonPropertyName"]` gets mapped to `user.name`.

### 第二个参数

第二个参数很关键，他是表示`jsonObject`中的字段与类中的字段的映射关系。意思是这个字段在`jsonObject`和类中的对应关系，即字段在类中字段从转换为json格式的类型中转化来的。第二参数时选填的，默认为`Any`。

> The second parameter of `@JsonProperty` describes what happens when doing the mapping between JSON and TypeScript objects. This parameter is optional; the default value is `Any` (which means no type check is done when the mapping happens).

我大概将映射关系区分为三种：

第一种是**基本数据类型映射**

```typescript
@JsonProperty("jsonPropertyName", String)
name: string = undefined;
```

基本数据类型其实可以理解ts官方中帮你实现了对应关系转换，基本数据类型映射主要是ts中几种基本数据类型与属性对应的关系，例如string、number此类的。当`JSONObject`中的类型与ts所给到基本类型可以直接对应时，可以使用插件提供的映射关系。

| expected type | typescript type |
| ------------- | --------------- |
| String        | string          |
| Number        | number          |
| Boolean       | boolean         |
| User          | User            |
| Any           | any             |
|               |                 |
| [String]      | string[]        |
| [Number]      | number[]        |
| [Boolean]     | boolean[]       |
| [User]        | User[]          |
| [Any]         | any[]           |

> 注意：在我们的项目实际使用中，因为common打包的关系，不能明确表示的Obj类型，不能使用any，应该使用Object。例如Map



第二种是**对象类型**

对象类型也是非常简单的，其实就是由我们在项目中直接定义的类，例如上一个例子中的 `User`类。这种类型主要是出现在转换的类型中，包含一些子类型。

```typescript

@JsonObject
export class User {
    @JsonProperty("jsonPropertyName", Name)
    name: Name = undefined;
}

@JsonObject
class Name {
 	@JsonProperty("jsonPropertyFirstName", String)
    firstName: string = undefined;
  
    @JsonProperty("jsonPropertyLastName", String)
    lastName: string = undefined;
}
```

当你将子类也使用`JsonConvert`相关的装饰器装饰后，内部的子类也会被递归序列化出来。



第三种是**自定义Converter类型**

第三种其实是一个很关键的类型，在业务场景中，很多时候后端传给我们的对象并不是我们前端所需要的，我们有时候需要对某个字段进行重新定义，或者根据前端业务的业务场景重新转换后才能使用。例如，我们依然用`User`作为例子。

```typescript
this.http.get<User>('/api/users')
 .subscribe(user => {
  console.log(user); // {name: Tom Smith}
});
```

后端传给我们的`User`中的`name`属性的值是`Tom Smith`，是姓名连在一起的。但是在前端的业务中，我们却有一个`Name`类型来维护姓名。

```typescript
export class Name {
  // 名
  firstName: string;
  // 姓氏
  lastName: string;
}
```

当我们在序列化的时候，其实我们是希望能够直接可以将后端传过来的`name`直接序列化为`Name`类型的，但是这种情况，并不是第二种对象类型我们无法直接将`name`序列化为`Name`类型的，所以我们需要使用插件为我们提供的自定义Converter。

```typescript
@JsonConverter
class NameConverter implements JsonCustomConvert<User> {
    serialize(name: Name): string {
        return name.firstName + ' ' + name.lastName;
    }
    deserialize(name: any): Name {
        return new Name('firstName', 'lastName');
    }
}
```

首先我们需要定义一个继承自`JsonCustomConvert`接口的类，并实现内部的序列化和反序列方法，而序列化反序列化所设计的数据处理逻辑则可以直接写在**converter的方法中**，这样就可以避免每次获取数据后调用类似于**initName()**之类这样的方法。

```typescript
@JsonObject
export class User {
    @JsonProperty("name", NameConverter)
    name: Name = undefined;
}
```

然后呢，我们只需要将第二个参数改为我们实现了`JsonCustomConvert`接口的**Converter**类就可以了。

### 第三个参数

`@JsonProperty`的第三个参数也是可选的，他是一个`boolean`值，意思是所序列化的对象，在原`JSONObejct`对象中该字段是否能不存在。默认为`false`，也就是说，默认这个字段是必须存在的，不然会抛出一个异常。

> The third parameter of `@JsonProperty` determines whether the `jsonProperty` has to be present in the json. This parameter is optional; the default value is false.



## 实例化JsonConvert并调用相关方法

```typescript
// jsonObj是通过 JSON.parse() 得到的json对象 
// Now you can map the json object to the TypeScript object automatically
let jsonConvert: JsonConvert = new JsonConvert();
let user: User = jsonConvert.deserializeObject(jsonObj, User);
console.log(user); // prints User{ ... } in JavaScript runtime, not Object{ ... }
```

> Tip: All `serialize()` and `deserialize()` methods may throw an exception in case of failure. Make sure you catch the errors in production!

通过上例我们可以看到在调用`jsonConvert.deserializeObject()`时我们需要先实例出一个`JsonConvert`的对象，但其实在项目中我们并不需要每次使用时都去维护这样一个对象实例。所以我们可以选择在项目通过单例模式将`JsonConvert`给包装一下，这样可以省去重复构建创建对象的开销。

当我们实例化`JsonConvert`后，`json2typescript`提供了一些属性可以配置。

| 属性名                   | 作用                                       | 配置项                                      |
| --------------------- | ---------------------------------------- | ---------------------------------------- |
| operationMode         | Determines how the JsonConvert class instance should operate. | `OperationMode.DISABLE`: json2typescript will be disabled, no type checking or mapping is done；<br />   `OperationMode.ENABLE`: json2typescript is enabled, but only errors are logged；<br />   `OperationMode.LOGGING`: json2typescript is enabled and detailed information is logged |
| valueCheckingMode     | Determines which types are allowed to be null | `ValueCheckingMode.ALLOW_NULL`: All given values can be null；  <br />`ValueCheckingMode.ALLOW_OBJECT_NULL`: Objects can be null, but primitive types cannot be null  ；<br />`ValueCheckingMode.DISALLOW_NULL`: No null values are tolerated |
| ignorePrimitiveChecks | Determines whether primitive types should be checked | True:  it will be allowed to assign primitive to other primitive types.<br />default false; |

所以我们可以根据我们项目的需要，在新建实例实例的进行配置下面是项目中的配置图：

![JsonConvert类配置](/img/17-12/18-2.png)



# 思考

 在实际项目中使用`json2typescritp`可以使用其为我们提供的`Conveter`来实现我们自定义的一个映射关系，但是这样使用时我们也不可避免会出现一些问题。下面是我所考虑到一些需要注意的事情。

##  尽量避免显示init()

经常当我们拿到一个类之后，由于前后端数据结构有一定的差异，所以我们在拿到后端的数据后会在回调函数里面去调用一些类似`init()`的方法，现在我们可以将这种单纯的数据逻辑的转型放到`Conveter`里面。



## 序列化时的幂等问题

幂等这个问题出现的场景很常见，主要是当我们可能会对一个对象进行重复的序列化的时候。如果我们在使用`@JsonPropery`的时候，没有使用自定义的**Conveter类型**，就不需要考虑幂等的问题，因为只使用**对象类型**和**基础数据类型**是不会有幂等问题的。

但是当我们使用自定义了Conveter转换的时候就需要注意这种情况。还是以上面的`User`类作为例子：

```typescript
@JsonConverter
class NameConverter implements JsonCustomConvert<User> {
    serialize(name: Name): string {
        return name.firstName + ' ' + name.lastName;
    }
 	// 在反序列化时，是将name转换为Name的实例
    deserialize(name: string): Name {
        return new Name('firstName', 'lastName');
    }
}

@JsonObject
export class User {
    @JsonProperty("name", NameConverter)
    name: Name = undefined;
}
```

我们先如上图定义了这样一个`User`的类型，当我们第一次序列化时由于是用的后端传来的对象，`name`字段确实是`string`类型的，这个时候`user`就可以被成功初始化出来。

```typescript
// 成功初始化
let user: User = jsonConvert.deserializeObject(jsonObj, User);

// 这里假设一系列的业务处理
...

// 第二次初始化： error
let user: User = jsonConvert.deserializeObject(jsonObj, User);
```

当我们第二次初始化的实话却报错了，因为 `name` 属性在第一次初始化后已经成为了`Name`类型，第二次的时候依然会将其当做`string`类型处理，自然就报错了。

这里我想到两种解决方案，第一种是直接将后端属性进行隔离，使用这种方案的话就可以在每次反序列化时都是通过有后端获取的`name`属性来操作的：

```typescript
@JsonObject
export class User {
    // 保留后端的字段，但其不参与业务
    @JsonProperty("name", String)
    private name: string = undefined;
  
    @JsonProperty("name", NameConverter)
  	// 前端使用的字段，通过name转换而来
  	userName: Name = undefined;
}
```

第二种方案则是在反序列化的方法中做处理： 

```typescript
@JsonConverter
class NameConverter implements JsonCustomConvert<User> {
    serialize(name: Name): string {
        return name.firstName + ' ' + name.lastName;
    }
 	
    deserialize(name: any): Name {
      	// 加入判断，如果name已经是实例了则直接返回
      	if(name instanceof Name) {
        	return name;	
      	}
        return new Name('firstName', 'lastName');
    }
}
```



## Map与Set的序列化和反序列化

虽然`json2typescript`可以在前后端通信时帮助我们序列化对象，但是在前端业务中也可以帮到我们。

首先我们知道`es6`中有一个坑，那就是`JSON.stringify()`是无法处理`Map`和`Set`类型的。对于这两种类型直接序列化返回的结果都是`{}`。

```typescript
JSON.stringify(new Map([['a',2], ['b',3]])) // {}
```

由于Map和Set都是es6中提供的类型，`JSON.stringify()`并没有提供支持，我们也可以通过使用`json2typescript`来提供这种扩展。

```typescript
@JsonConverter
class MapConverter implements JsonCustomConvert<Map> {
    serialize(map: Map): any {
        return [...map];
    }
 	// 在反序列化时，是将name转换为Name的实例
    deserialize(map: any): Map {
        return new Map(...[name]);
    }
}

@JsonObject
export class User {
  	// 注意这个时候需要设置第三个参数因为他不是一定存在的
    @JsonProperty("map", MapConverter, true)
    map: Map<string, string> = undefined;
}
```

并且我们也经常会遇到后端常常使用`HashMap`或者`HashSet`类型，我们也可以使用`自定义Conveter`来将其在前端也转为Map。