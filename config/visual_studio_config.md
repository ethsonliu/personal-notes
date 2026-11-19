### 插件

  1. `RemoveTrailingWhitespaces`，去除末尾的空格
  2. `Visual Assist`，官网 https://www.wholetomato.com/
  3. `Code Alignment`，提供代码对齐功能, 比如按 =, (, ) 等符号对齐, 也可以自定义对齐基准字符(串)
  4. `Indent Guides`，给每个代码块增加垂直对齐虚线

### 配置 deepseek

用插件的话 remote docker 时无法使用，暂未找到原因，可以手动添加 ds 来解决避免安装插件。

打开 chat 里模型选择列表，点击 + 号，Add Models，然后 Custom Endpoint，输入 group name，填 deepseek 就行，然后是 api key，接着选择 Chat Completions，接着进入 chatLanguageModels.json，

```
{
		"name": "deepseek",
		"vendor": "customendpoint",
		"apiKey": "${xxxxxx}",
		"apiType": "chat-completions",
		"models": [
			{
				"id": "deepseek-v4-pro",
				"name": "deepseek-v4-pro",
				"url": "https://api.deepseek.com/chat/completions",
				"toolCalling": true,
				"vision": false,
				"maxInputTokens": 655360,
				"maxOutputTokens": 393216
			}
		]
	}
```

