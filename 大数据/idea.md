# Idea使用技巧

用了idea，你可以丢掉Sqlyog，workbeanch，navicat，postman，xshell，securtCRT等工具。

## Basic Settings

修改全局设置，换行符，文件编码，Deployment，Maven，Compiler，Terminal，SSH Terminal

![image-20200320213453535](image\file-settings-project.png)

1. 换行符![image-20200320214015867](image\换行符设置.png)

2. 文件编码![image-20200320214101119](image\文件编码.png)

3. Deployment（连接远程服务器）![image-20200320214211322](image\Deployment.png)

4. Maven（换成自己的）![image-20200320214257632](image\Maven.png)

   ![image-20200320214446240](image\maven-local-repo.png)

   ![image-20200320214517667](image\maven-mirrors.png)

   ![image-20200320214556119](image\maven-profile.png)

5. Compiler And Annotation Processors![image-20200320214657133](image\java-compiler.png)

   ![image-20200320214804317](image\annotation-processors.png)

6. Spring配置文件提示![image-20200320214913360](image\spring-config.png)

7. Terminal换成Git![image-20200320215032252](image\terminal.png)

8. SSH Terminal字符编码![image-20200320215104291](image\ssh-terminal.png)

9. 自动导入包![image-20200320215154815](image\auto-import.png)

## Database

配置连接数据库

1. ctrl shift a弹出对话框![image-20200320215355317](image\database-dailog.png)
2. alt + ins添加数据源，点击download下载驱动，可以自己下载驱动![](image\add-datasource.png)

## ![image-20200320215643063](image\database-config.png)

3. 代码有提示哦输入ins upd sel，tab键。利用快捷键提示，可以弹出表面，字段名。![image-20200320215935196](image\sel.png)



## SFTP

远程连接服务器，可以直接上传，下载文件，远程修改文件。

1. ctrl shift a弹出对话框![image-20200320220254260](image\remote-host-dailog.png)
2. 选择对应服务器，（Deployment配置才有）![image-20200320220403830](image\remote-host.png)
3. 修改文件，双击文件可直接修改，在idea中编辑文件是有提示的，还可以格式化，点这里上传，![image-20200320220545225](image\edit-remote-file.png)
4. 下载文件，选中文件，直接复制，然后再idea中粘贴就可以下载![image-20200320220725197](image\download-copy.png)![image-20200320220756044](image\download-paste.png)![image-20200320220824234](image\download-file.png)
5. 上传文件，在idea的项目中选中文件，复制，然后切到remote host，选中目录，粘贴就可以上传。![image-20200320220858470](image\upload-copy.png)![image-20200320220941853](image\upload-file.png)

## Docker

## HTTP

用来发送http请求，很方便，还可以测试，新建test.http文件。不懂得可以看案例，非常简单，复制过来就可以用。

![image-20200320221519982](image\http.png)

## Plugins

1. Keypromote X快捷键提示插件
2. Lombok Java代码减少插件
3. Alibaba Java Coding 阿里巴巴代码规范插件
4. GitToolBox Git提交模板插件

![image-20200320221410142](image\plugins.png)

## Shortcut

idea自带了快捷键文档，是英文版的，代码完成提示，需要改快捷键，不然与输入法切换有冲突，Windows下。

![image-20200320221604318](image\keymap.png)

## License

下次再说，使用Github开源项目免费申请JetBrains全家桶一年试用期，可以续期。

