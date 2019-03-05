title: 我们家CSCWFE+总结
date: 2017-05-07
tags: 娱乐+经验

---

## CSCWFE

> 新招收的一帮小伙子小姑娘还挺不错,怎么说在学习态度上还是挺让人欣慰的,希望这一个月的培训能给他们带来帮助,哈哈未来是你们的!
> 
> 一个月培训下来,自己在做准备工作的时候还是有一些收获,有些不常用忘掉的知识可以补回来,原有的知识还能加深理解,跟小伙子小姑娘们一起的这段时间是非常快乐的~再详细的东西就记在心中,不多说啦.

        
## 总结

### 中秋佳节
**项目FE技术:** 中秋活动采用vuejs框架,使用gulp webpack打包,gulp livereload的自动刷新功能对开发提供了很大的便利,vue以组件切换的形式代替了传统的页面跳转刷新实现视图的切换.
**项目目的:** 为了更好的卖月饼O(∩_∩)O!!!购买月饼的客户可拥有发起祝福的机会,通过祝福的分享点赞列出排行,从而通过排名送出活动礼品.
**项目技术内容分析+收获:**


1.由于原先有写过vue项目,所以对vue的开发还是有点经验的,对于一些小细节上的问题还是能注意到,但是这次是完全的自己写的项目,出现的问题还是不少,收获很大.

2.样式的问题,由于使用vue是将项目所有的js文件打包成一个js,所有的style打包在一起,所以不同组件的class名需要区分开来,不然虽然style写在不同组件,但由于最后是打包在一块,所以是一起加载的,会影响各个组件的样式.

3.逗号问题,在vue中逗号也是一个需要时刻注意的小细节,在每一个变量,每一个函数,每一个块后边都习惯性的加一个逗号,否则会报错,虽然在终端能看到错误的位置,但是注意好这些细节还是能节约开发时间,时间是宝贵的,虽然看起来似乎不多,但给自己少制造麻烦是个好习惯.

4.routerjs

    "/receive": {
    	component: receive
    },
    "/receiveList/:userId": {
    	name: 'userList',
    	component: receiveList
    },
    "/blessingDetail/:wishId/:state": {
    	name: 'blessingDetail',
    	component: blessingDetail
    },

    /receive路由路径名,name此路由(表示整个路由)名字,component此路由加载的组件,组件的切换位置对应相应的router-view标签内,/:wishId表示对路由传入的数据,在路由中接收,router.go()或v-link需带上params对象,另外还有subRoutes表示此路由的子路由,路由可层层递进.


5.background,background-size:100% 100%可使背景图在当前区域完全拉伸显示出来,但是这会有点输入框弹出输入法将背景往上挤压的问题,通过background-size:cover可以解决挤压问题,cover是让背景覆盖在区域上,但是cover在图片较大时会使图片显示不全,background-attachment:fixed可使背景固定不动,而默认的scroll表示背景会随着内容的滚动的滚动上去.

6.cookie设置记录登录状态

    var setCookie = function(name, value, expires, path, domain, secure){
    	var cookieText = encodeURIComponent(name) + "=" + encodeURIComponent(value);
    	var cookieDate = new Date();
    	cookieDate.setDate(cookieDate.getDate() + expires);
    	// if(expires instanceof Date) {
    	// 	cookieText += "; expires=" + expires.toGMTString();
    	// }
    	cookieText += "; expires=" + cookieDate.toGMTString();
    	if(path) {
    		cookieText += "; path=" + path;
    	}
    	if(domain) {
    		cookieText += "; domain=" + domain;
    	}
    	if(secure){
    		cookieText += "; secure";
    	}
    	document.cookie = cookieText;
    };
    
     	setCookie("phone", me.telephone, 30);
    
    	var getCookie = function(name){
    		var cookieName = encodeURIComponent(name) + "=";
    		var cookieStart = document.cookie.indexOf(cookieName);
    		var cookieValue = null;
    		if(cookieStart > -1) {
    			var cookieEnd = document.cookie.indexOf(";" , cookieStart);
    			if(cookieEnd == -1) {
    				cookieEnd = document.cookie.length;
    			}
    			cookieValue = decodeURIComponent(document.cookie.substring(cookieStart + cookieName.length, cookieEnd));
    		}
    		// console.log(cookieValue);
    		// this.telephone = '123';
    		return cookieValue;
    	};
    	
    	if(getCookie("phone") !== null){
    		this.$route.router.go({
    			name: "userList",
    			params: {
    				userId : getCookie("phone")
    			},
    		});
    	}
	

7.验证码倒计时:event为触发的函数的参数,触发时无需传入

    var time = 60;
    var timer;
    timer = setInterval(function(){
    	if(time > 0){
    		this.retimer = true;
    		event.target.innerHTML = "还有 " + (time--) + " 秒";
    		event.target.disabled = true;
    	} else {
    		this.retimer = false;
    		window.clearInterval(timer);
    		event.target.innerHTML = "获取验证码";
    		event.target.disabled = false;
    	}
    }, 1000);



8.变量的作用域,全局的变量可以存在data中,使用this关键字访问.

9.

> 最后也是最大的boss,微信的api,原先以为只是简单的调用api,谁知是个大坑.微信的api还是有很多不完善的地方,签名上头一步错,后面一直没法用,时间就这么拖着,当可以使用了又有很多使用上的困难,在每一个需要使用微信接口的页面都需要对微信接口进行配置.
> 
> 比如,想要听取微信的接收的录音,由于不是发出端,所以没有发送的本地id可以在微信听取录音中直接调用,这样的话就需要另外写个播放器来听取录音.又由于微信录取的录音是arm格式,h5中audio不支持arm格式,找了很久的转码方式,最后还是后台那边转码传过来才能听取.
> 
> 另外,需要特别注意的一点是,想要微信接口支持iphone,在加入
> 
>     <script src="http://res.wx.qq.com/open/js/jweixin-1.0.0.js"></script>
> 
> 后还需要另外加入
> 
>     <script src="https://res.wx.qq.com/open/js/jweixin-1.0.0.js"></script>
> 
> 而且,还没有探究出来的一个bug就是在多个页面上,有的写了禁止分享操作,有的写了分享操作,在Android上是可以正常执行的,但是在iPhone上会导致这几个页面全部都没有分享,这是项目正常运行的一个重要的细节,当时这问题是通过绕过问题来跳过了,但是绕过问题而不是解决的问题的方法是不支持的,这个问题还值得探究探究.
> 
> 总之,微信api坑不少,可能以后还会遇到其他的,慢慢积累经验吧~


