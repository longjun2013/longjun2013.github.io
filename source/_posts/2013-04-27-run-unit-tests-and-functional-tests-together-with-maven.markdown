---
layout: post
title: "使用Maven工具让Web应用一键运行Unit Tests和Functional Tests"
date: 2013-04-27 23:40
comments: true
categories: ["Maven", "构建脚本","Unit Tests" ,"Functional Tests", "AngularJS", "Karma"]
---

最近在做Web项目开发，使用AngularJS。一开始不知道怎么做测试，后来，在[这个页面](http://www.yearofmoo.com/2013/01/full-spectrum-testing-with-angularjs-and-testacular.html)里了解了很多AngularJS的单元测试和功能测试(End to End测试)技巧。 因此，在项目中，我们使用[Karma](http://karma-runner.github.io/0.8/index.html)(之前叫做Testacular)来运行Javascript的单元测试，也用Karma来运行AngularJS的e2e测试。

接着，我们写了两个Karma的配置文件，一个是单元测试的，一个是e2e测试的。我们可以用Karma命令单独运行这两种测试，但是我们又希望能够把这两个测试集成到应用的构建脚本中，以便我们可以运行`mvn clean install`就运行这两种测试，只要有一种测试失败，构建就失败。
