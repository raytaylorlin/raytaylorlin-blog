title: 使用Sublime编辑器打印高亮代码
date: 2017-12-21 15:36:29
categories:
- 技术
- 工具
- IDE
tags:
- Sublime
- 打印高亮代码
---
Sublime Text 3编辑器的代码高亮（主题）效果不错，本文介绍如何使用ST3将其打印成PDF。

<!-- more -->

# Sublime Text 3打印代码

ST3打印代码的原理是将代码（含样式）先转换成HTMl，再从浏览器中打印。

- 安装[ExportHtml](https://github.com/facelessuser/ExportHtml)插件
- 菜单`Preferences->Package Settings->ExportHtml->Settings`，在右侧添加如下自定义设置并保存


    {
        "html_panel": [
            {
                "Browser Print - Custom": {
                    "numbers": true,
                    "wrap": 900,
                    "browser_print": true,
                    "multi_select": true,
                    "color_scheme": "Packages/Color Scheme - Default/Monokai.tmTheme",  // 使用ST默认的Monokai主题，这里可以换成自己安装的其他主题路径
                    "style_gutter": true,
                    "diable_nbsp": true
                }
            }
        ]
    }


- `Ctrl-Shift-P`，`Export To HTML: Show Export Menu`选择上面自定义的选项，这时会弹开浏览器的打印页，HTMl已经生成
- 按下图设置打印选项，可以打印得更美观

![Chrome浏览器打印设置](https://raytaylorlin-blog.oss-cn-shenzhen.aliyuncs.com/image/IDE/Chrome%E6%B5%8F%E8%A7%88%E5%99%A8%E6%89%93%E5%8D%B0%E8%AE%BE%E7%BD%AE.png)

## 优化选项

默认导出的HTML会带文件名，如果想要将其隐藏，可以修改插件的导出CSS：

- 找到ExportHtml插件的路径：`Sublime Text Build 3143 x64\Data\Installed Packages\ExportHtml.sublime-package`，将插件文件当做压缩包解压出来
- 将解压出来的`ExportHtml\css\export.css`复制到其他地方，例如`Sublime Text Build 3143 x64\Data\Packages\User`，在文件末尾加上一行代码：

    #file_info { display: none; }

修改上面的插件设置，使用自定义的CSS：

    {
        "export_css": "Packages/User/export.css",  // 这里填自定义的CSS路径
        "html_panel": [
            // 保持原样，省略......
        ]
    }

重新导出，会发现烦人的文件名不显示了。