# compress_pack
> YUI Compressor Maven、jcv-maven-plugin、maven执行node压缩打包和fis3

## 1.YUI Compressor Maven
> warSourceExcludes是在编译周期进行完成后从src/main/webapp目录复制时忽略的文件packagingExcludes是在复制webapp目录完成后打包时忽略的文件
```
<plugin>
    <groupId>net.alchim31.maven</groupId>
    <artifactId>yuicompressor-maven-plugin</artifactId>
    <version>1.5.1</version>
    <executions>
        <execution>
            <goals>
                <goal>compress</goal>
            </goals>
        </execution>
    </executions>
    <configuration>
        <!-- 读取js,css文件采用UTF-8编码 -->
        <encoding>UTF-8</encoding>
        <!-- 不显示js可能的错误 -->
        <jswarn>false</jswarn>
        <!-- 若存在已压缩的文件，会先对比源文件是否有改动  有改动便压缩，无改动就不压缩 -->
        <force>false</force>
        <!-- 在指定的列号后插入新行 -->
        <linebreakpos>-1</linebreakpos>
        <!-- 压缩之前先执行聚合文件操作 -->
        <preProcessAggregates>true</preProcessAggregates>
        <!-- 压缩后保存文件后缀 无后缀 -->
        <nosuffix>true</nosuffix>
        <!-- 源目录，即需压缩的根目录 -->
        <sourceDirectory>src/main/webapp</sourceDirectory>
        <!-- 压缩js和css文件， 可不需要，默认全部 -->
        <includes>
            <include>static/**/*.js</include>
            <include>static/**/*.css</include>
        </includes>
        <!-- 以下目录和文件不会被压缩， 可不需要，默认不排除 -->
        <excludes>
            <exclude>static/**/*.min.css</exclude>
        </excludes>
        <!-- 以下可不需要 -->
        <!-- 压缩后输出文件目录 -->
        <outputDirectory>${basedir}/mobile</outputDirectory>
        <!-- 聚合文件 -->
        <aggregations>
          <aggregation>
            <!-- 合并每一个文件后插入一新行 -->
            <insertNewLine>true</insertNewLine>
            <!-- 需合并文件的根文件夹 -->
            <inputDir>${basedir}/mobile/scripts</inputDir>
            <!-- 最终合并的输出文件 -->
            <output>${basedir}/mobile/scripts/app/app.js</output>
            <!-- 把以下js文件合并成一个js文件，是按顺序合并的 -->
            <includes>
              <include>app/core.js</include>
              <include>app/mlmanager.js</include>
            </includes>
          </aggregation>
        </aggregations>
    </configuration>
</plugin>
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-war-plugin</artifactId>
    <configuration>
        <warSourceExcludes>static/**/*.js,static/**/*.css</warSourceExcludes>
    </configuration>
</plugin>
```

## 2.jcv-maven-plugin
```
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-war-plugin</artifactId>
    <configuration>
        <warSourceExcludes>**/*.js,**/*.css</warSourceExcludes>
    </configuration>
</plugin>
<plugin>
    <groupId>com.iqarr.maven.plugin</groupId>
    <artifactId>jcv-maven-plugin</artifactId>
    <version>1.0.0</version>
    <executions>
        <execution>
            <id>process</id>
            <phase>prepare-package</phase>
            <goals>
                <goal>process</goal>
            </goals>
        </execution>
    </executions>
    <configuration>
        <!--需要处理的文件后缀-->
        <suffixs>
            <param>html</param>
        </suffixs>
        <!--清理页面注释-->
        <clearPageComment>true</clearPageComment>
        <!-- 压缩css-->
        <compressionCss>true</compressionCss>
        <!-- 压缩js-->
        <compressionJs>true</compressionJs>
        <!-- 全局js 方法-->
        <globaJsMethod>MD5_METHOD</globaJsMethod>
        <!-- 全局css方法-->
        <globaCssMethod>MD5_METHOD</globaCssMethod>
    </configuration>
</plugin>
```
### 注意事项
- 不支持 ../../xxx.js
- 不支持 ../../xx.css
- 如果启用js压缩，那么在js中变量定义禁止使用js关键字
- html 清除注释只支持网页中的
- 插件不会在eclipse中生效，在package后才会生效
- 注意在使用md5文件名的时候请注意排除一些js动态加载css,如果修改了文件名会导致无法加载到css,因此需要排除掉，目前已知有kindeditor,layer,My97DatePicker
- js css文件编码必须utf-8

## 3.上述两种方式需要替换字符插件配合
```
<plugin>
    <groupId>com.google.code.maven-replacer-plugin</groupId>
    <artifactId>replacer</artifactId>
    <version>1.5.3</version>
    <executions>
        <execution>
            <phase>prepare-package</phase>
            <goals>
                <goal>replace</goal>
            </goals>
        </execution>
    </executions>
    <configuration>
        <basedir>${build.directory}</basedir>
        <includes>
            <include>${build.finalName}/**/*.html</include>
        </includes>
    <replacements>
        <replacement>
            <token>href=\"/static/</token>
            <value>href=\"\$!staticResourcesUrl/static/</value>
        </replacement>
            <replacement>
                <token>src=\"/static/</token>
                <value>src=\"\$!staticResourcesUrl/static/</value>
            </replacement>
        </replacements>
    </configuration>
</plugin>
```

## 4.maven里面执行node命令npm
> 利用maven插件exec-maven-plugin执行npm命令来执行package.json文件

### package.json文件
```
{
  "scripts": {
    "start": "npm install -g fis3 && npm install -g fis-parser-less && npm install -g fis3-postpackager-loader && npm install -g fis3-deploy-replace && npm install -g fis-optimizer-html-compress && fis3 release -d ../webapp",
    "build": "fis3 release -d ../webapp"
  }
}
```
### pom文件
```
<plugin>
  <groupId>org.codehaus.mojo</groupId>
  <artifactId>exec-maven-plugin</artifactId>
  <executions>
    <execution>
      <id>exec-npm-install</id>
      <phase>prepare-package</phase>
      <goals>
        <goal>exec</goal>
      </goals>
      <configuration>
        <executable>npm</executable>
        <arguments>
          <argument>build</argument>
        </arguments>
        <workingDirectory>${basedir}/src/main/webapp</workingDirectory>
      </configuration>
    </execution>
    <execution>
      <id>exec-npm-run-build</id>
      <phase>prepare-package</phase>
      <goals>
        <goal>exec</goal>
      </goals>
      <configuration>
        <executable>npm</executable>
        <arguments>
          <argument>run</argument>
          <argument>build</argument>
        </arguments>
        <workingDirectory>${basedir}/src/main/webapp</workingDirectory>
      </configuration>
    </execution>
  </executions>
</plugin>
```
### fis-conf.js
```
// 加 md5,先去掉就不用改路径
fis.match('*.{js,css}', {
    useHash: true
});
// 启用 fis-spriter-csssprites 插件
fis.match('::package', {
    spriter: fis.plugin('csssprites')
});
// 对 CSS 进行图片合并
fis.match('*.css', {
    // 给匹配到的文件分配属性 `useSprite`
    useSprite: true
});
fis.match('*.js', {
    // fis-optimizer-uglify-js 插件进行压缩，已内置
    optimizer: fis.plugin('uglify-js')
});
fis.match('*.css', {
    // fis-optimizer-clean-css 插件进行压缩，已内置
    optimizer: fis.plugin('clean-css')
});
fis.match('*.png', {
    // fis-optimizer-png-compressor 插件进行压缩，已内置
    optimizer: fis.plugin('png-compressor')
});
/**
fis.match('*.html', {
  //invoke fis-optimizer-html-minifier
  fis-optimizer-minify-html
  optimizer: fis.plugin('html-minifier')
});

fis.config.set('modules.optimizer', {
  html: 'minify-html'
});

fis.match('*.html', {
  optimizer: fis.plugin('html-compress')
});
 **/
fis.match('**', {
    deploy: replacer([
        {
            from: 'href="/static/',
            to: 'href="${staticResourcesUrl}/static/'
        },
        {
            from: 'src="/static/',
            to: 'src="${staticResourcesUrl}/static/'
        }
    ]).concat(fis.plugin('local-deliver'))
})
function replacer(opt) {
    if (!Array.isArray(opt)) {
        opt = [opt];
    }
    var r = [];
    opt.forEach(function (raw) {
        r.push(fis.plugin('replace', raw));
    });
    return r;
};
/**不需要压缩、合并图片、也不需要 hash
 fis.media('debug').match('*.{js,css,png}', {
    useHash: false,
    useSprite: false,
    optimizer: null
})
 fis.match('**', {
    deploy: [
        fis.plugin('replace', {
            from: 'href="/static/',
            to: 'href="${staticResourcesUrl}/static/'
        }),
        fis.plugin('local-deliver') //must add a deliver, such as http-push, local-deliver
    ]
});
 **/
```
