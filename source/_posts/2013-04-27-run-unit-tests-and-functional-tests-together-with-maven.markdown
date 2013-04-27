---
layout: post
title: "使用Maven工具让Web应用一键运行Unit Tests和Functional Tests"
date: 2013-04-27 23:40
comments: true
categories: ["Maven", "构建脚本","Unit Tests" ,"Functional Tests", "AngularJS", "Karma"]
---

最近在做Web项目开发，使用AngularJS。一开始不知道怎么做测试，后来，在[这个页面](http://www.yearofmoo.com/2013/01/full-spectrum-testing-with-angularjs-and-testacular.html)里了解了很多AngularJS的单元测试和功能测试(End to End测试)技巧。 因此，在项目中，我们使用[Karma](http://karma-runner.github.io/0.8/index.html)(之前叫做Testacular)来运行Javascript的单元测试，也用Karma来运行AngularJS的e2e测试。

接着，我们写了两个Karma的配置文件，一个是单元测试的，一个是e2e测试的。我们可以用Karma命令单独运行这两种测试，但是我们又希望能够把这两个测试集成到应用的构建脚本中，以便我们运行

```
$ mvn clean install
```

时，就会运行这两种测试，只要有其中一种测试失败，构建就失败。

使用[Karma Maven Plugin](https://github.com/karma-runner/maven-karma-plugin)可以很轻松地配置单元测试和e2e测试。

配置如下：

<!-- more -->

{% codeblock lang:xml %}
<plugin>
	<groupId>com.kelveden</groupId>
	<artifactId>maven-karma-plugin</artifactId>
	<version>1.0</version>
	<executions>
    	<execution>
			<id>unit-test</id>
			<phase>test</phase>
            <goals>
				<goal>start</goal>
            </goals>
			<configuration>
        		<configFile>/path/to/karma.conf.unit.js</configFile>
    		</configuration>
        </execution>
        <execution>
			<id>e2e-test</id>
			<phase>integration-test</phase>
            <goals>
                <goal>start</goal>
            </goals>
			<configuration>
        		<configFile>/path/to/karma.conf.e2e.js</configFile>
    		</configuration>
        </execution>
    </executions>
</plugin>
{% endcodeblock %}

以上代码指定了单元测试和功能测试分别在Maven的Life cycle的`test`和`integration-test`阶段运行。

好了，当我们运行

```
$ mvn clean install
```

时，单元测试可以正常运行，但是e2e测试却失败了。原来，AngularJS的e2e测试需要启动Web服务器，访问真实的URL才行，否则e2e测试就没有意义了呀。
由于我们还没有配置Web服务器，所以e2e测试就失败了。

那么，接下来就是配置Jetty服务器，让Jetty服务器在运行`integration-test`之前启动，在`integration-test`运行完成之后关闭就好了。同样，我们要用[Jetty Maven Plugin](http://docs.codehaus.org/display/JETTY/Maven+Jetty+Plugin)来达到目的。

下面是Jetty的配置：

{% codeblock lang:xml %}
<plugin>
        <groupId>org.mortbay.jetty</groupId>
        <artifactId>maven-jetty-plugin</artifactId>
        <version>6.1.10</version>
        <configuration>
                <stopKey>foo</stopKey>
                <stopPort>9999</stopPort>
        </configuration>
        <executions>
                <execution>
                        <id>start-jetty</id>
                        <phase>pre-integration-test</phase>
                        <goals>
                                <goal>run</goal>
                        </goals>
                </execution>
                <execution>
                        <id>stop-jetty</id>
                        <phase>post-integration-test</phase>
                        <goals>
                                <goal>stop</goal>
                        </goals>
                </execution>
        </executions>
</plugin>
{% endcodeblock%}

如上面的代码所示，Jetty会在Maven的Life Cycle中的`pre-integration-test`阶段启动，在`post-integration-test`阶段关闭。

好了，这下运行

```
mvn clean install
```

会先运行Unit Tests，再启动Jetty，然后再运行e2e测试，最后关闭Jetty服务器。

完美了。
