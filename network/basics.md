# Cookie & Session
客户想去银存钱，如果他是第一次去，那么银行就需要给他开户，办一张卡给他，这里银行就类似于server，客户就是client，client获得了银行给他的卡，那么下次他再去的时候，
就不需要重新办卡存钱了，这个卡就是这个人在银行的身份凭证，只要出示这个卡，银行就可以帮他存钱。

同时，银行这里会有这个人的账户，如果这个人存完钱了，还想要继续转钱，通常我们是在柜台（session）进行操作，在出示银行卡之后，柜台会暂时帮你进行操作，这时候客户就可以一直进行操作而不需要每次都出示银行卡之后才能进行操作。

另外，如果这时候这个人离开了，第二天再来重新进行操作，依旧需要重新出示银行卡，因为过了这么久，你和柜台建立的关系已经失效了，这时候需要重新出示银行卡。

## Cookie
当client向server发送请求的时候，server会创建一个cookie，并通过response发送给client，client会保存cookie，下次client发送的request里会附加上这个server的所有cookie,这时候server就会读取cookie里的信息，而不用每次都需要client发送所有信息。

Cookie的缺点：

1. Cookie数量和长度的限制。每个domain最多只能有20条cookie，每个cookie长度不能超过4KB，否则会被截掉。
2. 安全性问题。如果cookie被人拦截了，那人就可以取得所有的session信息。即使加密也与事无补，因为拦截者并不需要知道cookie的意义，他只要原样转发cookie就可以达到目的了。
3. 有些状态不可能保存在客户端。例如，为了防止重复提交表单，我们需要在服务器端保存一个计数器。如果我们把这个计数器保存在客户端，那么它起不到任何作用。

## Session

当client访问server时，server通过getSession方法创建或获得session，如果创建了一个session，server会同时创建一个jsessionId的cookie，里面包含了这个session的id，下次client发送请求时候，getsession会通过jsessionId获得对应的session。

session会针对不同浏览器持有多份，因此当你打开不同浏览器时，实际上不会保留上一个浏览器的内容，同样，携带不同的cookie也会

Session默认的生命周期是20分钟，可以手动设置更长或更短的时间。



