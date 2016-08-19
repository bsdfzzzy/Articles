##Babel解析es6模块系统

    在这里分析的内容基于babel配置如下：
    {
    	"presets": [
			"es2015",
	   		"react",
			"stage-2"
        ],
        "plugins":[],
    }

###最简单的export和import
我们创建两个文件，export.js和import.js。
首先，在export.js中写入如下代码：
    export const a = 'a'
在import.js中写入如下代码：
    import a from './export'
    console.log(a)