
# CherryRest
一种非常优雅Rest接口调用方式
## Features

- 基于Restful风格接口设计的模块化接口url生成器
- 集中管理接口url
- 默认使用fetch作为接口加载器
- 可以给模块单独设置baseUrl 
- 支持response formatter配置、支持单个module formatter
- 内置默认filter，当前只有一个over，重载response data filed


## Install

```bash
npm install cherry-rest --save
```

## Usage

```jsx
import Cherry,{module as CherryModule,config as CherryConfig} from 'cherry-rest'
```

基于业务Rest接口定义模块

```jsx
//方式一
CherryModule('name');

//方式二、定义多个模块、模块设置别名
CherryModule('module2|home,moudle3|user');

//方式三、复合继承模式
CherryModule([{
    name:'module3',
    alias:'about',
    children:[{
        name:'info',
    },{
        name:'concat'
    }]
}]);
```

调用

```jsx
Cherry.module3.info.query({query:{id:1}}).then(res=>res)
//fetch get /about/info?id=1;
Cherry.module3.info.create({body:{name:1}}).then(res=>res)
//fetch post /about/info {name:1};
Cherry.module3.info.update({query:{id:1}}).then(res=>res)
//fetch put /about/info?id=1;
Cherry.module3.info.remove({path:'1,2'}).then(res=>res)
//fetch delete /about/info/1/2;
```

Test

```js
import Cherry,{module as CherryModule,config as CherryConfig} from '../dist/index.js'
//http://ip.taobao.com/service/getIpInfo.php?ip=114.114.114.114
jest.setTimeout(1200000);

CherryConfig({
    credentials:'include'
},{
    baseUrl:'http://ip.taobao.com',
    formatter:function(res){
        return res.json()
    }
})

CherryModule('ip|service',{
    formatter:[function(data){
        return data.data;
    }],
    filters:[{
        name:'over',
        options:['city_id|id']
    }]
});

it('async & filters test', () => {
    expect.assertions(1);
    return Cherry.ip.query({path:'getIpInfo.php',query:{ip:'114.114.114.114'}}).then(data => {
        expect(JSON.stringify(data)).toEqual(JSON.stringify({"id":"320100"}))
    });
});
```

## Demo

### Base

```javascript
//import
import Cherry,{module,config} from 'cherry-rest'

//基于rest业务接口定义模块
//比如：
//http://localhost:8080/api/v1/user 
config({
    common:{
        formatter:function(res){
            return res.json()
        }
    }
})
module([{
    name:'api',
    children:[{
        name:'version',
        alias:'v1',
        children:[{
            name:'user'
        }]
    }]
}])

//也可以这么定义
config({
    common:{
        baseUrl:'/api/v1',
        formatter:function(res){
            return res.json()
        }
    }
})
module([{
    name:'user'
}])

let Loader=Cherry.user;

//调用
let Loader=Cherry.api.version.user;

Loader.create({body:{id:'001',name:'001'}}).then(res=>res);
//POST /api/v1/user {id:'001',name:'001'}
Loader.query({path:'001'}).then(res=>res);
//GET /api/v1/user/001
Loader.query({query:{name:'001'}}).then(res=>res);
//GET /api/v1/user?name=001;
Loader.update({path:'001',body:{name:'002'}}).then(res=>res);
//PUT /api/v1/user/001 {name:'002'}

```





