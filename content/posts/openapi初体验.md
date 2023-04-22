---
title: "OpenAPI初体验"
date: 2018-05-25T16:36:00+08:00
draft: false
---

问题的一开始源于客户和服务部门抱怨我的REST API文档写得不好，然后又了解到 django rest framework 利用 coreapi 能自动生成文档，再就是看到 swagger.io 上说得天花乱坠的，OpenAPI文档写完后，可以生成40种语言的客户端代码（用户都不用文档了，代码都生成了！！），外加N种服务端stub代码，另外演示文档真心漂亮。于是我开始了研究 REST API specification的各种语言了，这里简单总结备忘下。


API specification
-----------------


API sepecification有很多种语言，主流的有3种


* OpenAPI (swagger)
* RAML
* API blueprint


我最开始研究的是openapi，没写几个endpoint我就放弃了，层次太深，缩进了几层之后，完全不知道自己在什么地方了。


我马上转去研究 api blueprint，这个挺好，有专门的emacs apib-mode，而且层次没有那么深，看起来更直白一点，甚至自己搞了 MSON 的规范，总之写起来更像是给人看的，而不是给机器看的。apib的工具相对较少，好在可以转成 openapi，然后再生成代码等等 。不过，好景不长，稍微复杂一点的API表达能力就有限了（需要复制、粘贴），表达得也不是那么直白。


以我python程序员的角度看问题，这种东西应该由python来写，每个可利用的模块定义成function或class，然后引用一点，这样也可以将api spec拆分成小块，又要看，又好维护。事实上，确定有一个叫 openapi 的package，完成了这件事情，可以用它来写，不过只支持 2.0，就没有仔细研究了。


我没有再去研究RAML，因为它也是yaml，所以层次问题肯定也存在，我去试也下基于Eclipse的KaiZen编辑器，层次问题仍然存在，虽然有outline、补全提示和template，但层次问题仍然不好解决。据说 jetbrains 平台也有类似的插件，我没有去试。


正当我闹心的时候，我搜索到了这么一篇[文章](https://azimi.me/2015/07/16/split-swagger-into-smaller-files.html)，说的是使用json ref来把swagger文档拆分成多个，虽然文章结尾给出的方法 json-refs resolve无法达到他说的那样，但是思想是好的，于是我自己花一天时间写了一个[工具](https://github.com/windtail/openapi-bundle)，实现了文章上说的方法，并且不会展开local ref，在openapi最后文档中仍然保留对schema的ref。


Swagger-tools
-------------


openapi号称有很多工具，并且官方网站上也一直在宣传最新的3.0标准，所以我自己选择了3.0，然后很快就掉坑里了（一会儿再说）。


先说下我的工作方式，我使用emacs管理一个全是yaml文件的工程，然后使用$ref把他们都link起来。一开始我每次合并成一个文档后，然后拿swagger-editor打开，看看哪里有错误，再改，但是这样非常得慢，因为swagger-editor是基于网页的，本地文件变了不会自动加载到编辑器中去，还得每次点。官方有个swagger-validator貌似只支持2.0的，即使支持3.0，估计我也搞不太明白，因为不习惯nodejs那套东西，终于发现python也有一个叫 openapi-spec-validator的东西，虽然做得粗糙了点，但是仍然是可以用的，很方便集成到我的工作流中，每次保存时合并到一个文件中，然后调用 validator 检查，非常好。


不管能不能搞明白nodejs，swagger-editor和swagger-ui是必须要使用的工具，幸好官方提供了docker container，然后我再写一个alias，就可以在bash中使用了，如下




```
alias swagger-ui='echo "http://localhost:8000" && docker run --rm --name swagger-ui -p 8000:8080 -e SWAGGER\_JSON=/app/openapi.yaml -v $PWD:/app swaggerapi/swagger-ui'
alias swagger-editor='echo "http://localhost:8080" && docker run --rm --name swagger-editor -p 8080:8080 swaggerapi/swagger-editor'
```


 这里假设了我合并之后的文档名称为 openapi.yaml


### 生成代码


当我很兴奋了写了很多endpoint之后，我想试试生成代码这个功能，很不幸，真的被坑了。swagger-codegen稳定版本居然不支持 3.0版本的openapi，只支持到2.0，抱着试一试的心态去看看了正在开发的swagger-codegen的snapshot，太让人失望了，只支持几种语言，居然没有python也没有go，大约都是java和js。


于是我没有办法，又去看了下2.0版本的openapi规范，真心不如3.0，着实没有学习的动力，万般无奈下，到处找3.0转2.0找工具，只有一两种可选，而且不是收费，就是不好使。最后终于被我在github上发现了api-spec-converter，可以把3.0转成2.0。在解决了如何不使用root用户全局安装npm module后，才把它安装上了。然后使用命令行




```
api-spec-converter -f openapi_3 -t swagger_2 openapi.yaml > swagger.json
```


 


就可以转换，我也以为就这样结束了呢，使用少的东西，坑自己多，由于2.0的表达能力不是很强，外加转换工具有BUG，所以我列举了下注意事项如下


* path不能有summary和description，这些应该放到operation里（即get/put/post等里）
* 只支持一个server
* components里只能有schema，不能有其他的，如parameters，responses
* parameter最好不要有local ref，api-spec-converter不能正确处理
* parameter应定义在operation级，即get/put/post下面，公共的parameter，工具处理不正确
* additionalProperties不支持，工具也没有自动地删除它
* readOnly的properties不能具有required属性


终于生成了2.0的spec，有空再试codegen的质量吧。坐等codegen支持3.0，或者哪天我自己研究个Python或Go的工具吧。


总结
--


不管怎样，swagger-ui 我还是很喜欢的，api自带swagger-ui这样的界面，集文档和尝试一身，对用户非常友好，即使不能生成代码，也是不错的选择。


