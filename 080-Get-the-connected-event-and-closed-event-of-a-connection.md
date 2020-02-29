# 获取会话的连接和断开事件

> __关键字__: 连接, 会话, 连接事件, 断开事件

## 通过 host builder 的方法 ConfigureSessionHandler 注册 session 的连接和断开处理代码

    builder.ConfigureSessionHandler((s) =>
        {
            // things to do when the session just connects
        },
        (s) =>
        {
            // things to do after the session closes
        });