tiktoken 为openai 专用的分词器，只能识别openai 自己的模型，其他未注册的模型会报错

解决方法：

使用fastchat 启动mebedding 的Openai服务的时候，不使用原模型名，把模型名改成gpt-4可以骗过tiktoken的内部检查机制
