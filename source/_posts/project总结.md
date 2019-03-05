title: project总结
date: 2017-05-07
tags: 经验

---

好久没有写博文了，这确实不是个好习惯，毕竟心得体会不会一直牢记心中。

好了，如题，介绍下我的小项目了～

**项目名称**：A公司微信公众号上的官方

**项目描述**：用轻量级js=>vuejs，通过gulp-webpack打包，部分内容切分出来另外用基本的html，css和js完成，为了使打包后的文件能较小，更加提高打开的性能，虽然路由使用了懒加载（做这个项目时vue-router的文档的懒加载会报错，具体解决办法见下文）。这个项目作为公司官网主要是展示公司的一些信息，比如公司介绍，公司的资讯，公司相关产品等

**项目职责**：前端开发，包括页面的实现还有跟后台的接口对接调试

**项目技术记录**：

**1.**项目目录还是老样子，可复用组件抽出，普通组件分文件夹放好，项目结构就清晰多了，虽然都会这么想，但是做可能会忘了，这也是一种经验。
 
**2.**vue-router的懒加载，查询了相关资料后使用了这种方法有效：

    "pashName": { 		
        component: function(resolve) {
            require.ensure(['componentPath'], function(){
    			resolve(require('componentPath')); 			}); 		  },
    },

**3.**关于有些页面可能内容无法铺满整个屏幕，而当内容多的时候又能铺满屏幕，这时候可能会困扰背景问题，当内容足够的时候背景就有，而当内容不够的时候呢？
    解决：可以设置所需元素的最小高度，让它适应你所需要的高度，这样当内容不够的时候可以正常显示，内容足够的时候也会撑开。

**4.**只修改页面参数（params）的路由跳转会有什么事发生呢？比如在某个信息页，它有它的id，而这个信息页可以查看其他相关信息，当点击其他的信息，想跳转并查看其他信息。
    答：很遗憾，并没有什么变化，当用router.go()或v-link改变了传入了的params时它只是改变了url，因为目前组件就是渲染的，并没有重新渲染一遍组件。
    解决：这时候就需要自己加入refresh事件，可能有很多朋友会想到用window.location.reloas()重新加载，当然这样也可以，但是这样会刷新页面，体验就没那么好了。我们可以将传入的params重新请求所需的信息然后更改data更新信息，这样可以不用刷新便可以将信息更新，当然这还需要加上$(window).scrollTop(0)让页面重新回到顶部。

**5.**嗯这是个大坑，把项目拖的非常的慢，打开也是极其的慢，那就是图片的问题，由于这个官网有较多的图片，当时全用的本地链接，导致项目打包出来的文件极其大，当访问的时候就请求js文件要10m左右，这在高网速的时候也要好几秒，甚至好几分钟的相应时间，这是极其不友好的，也非常影响调试效率。后来将文件通过放在‘牛牛’上用外链形式访问，网页加载速度就可以秒开了，这个问题当时也找了好久，最后解决的时候别提有多舒服了～～～谨记！！！
 

**6.**frozen的slider在页面上由于是要将背景设好的，在其他地方可能也可以，但是在vue上想要设置动态的背景图会报错，原因暂时还没有找出来（尴尬），解决当然就只能在js添加相应的li了：

    for(var i = 0; i < res.banner.length; i++) {
      $('#slider1 ul').append("<li><span></span></li>");
      var backgroundImage = res.banner[i].image_url;
      $('#slider1 ul li').eq(i).children().css('backgroundImage','url('+backgroundImage+')');
    }

**7.**接下来是重头戏啦，我们常用的page组件，虽然当时找了很多，但是由于各种原因，找不到比较合适的，后来有了一个page，此处只写引用方法了：
    
    页面中的引用方式，传入改变页面的函数，总页数，记录页数到session，当返回的时候记录当前页面。
    <page :total="total" :change-page="changePage" :applistpage="applistpage"></page>

    applistpage: sessionStorage.applistpage,

    changePage(page) {
        var that = this;
        $.ajax({
          type: 'get',
          url: that.Host+'/Application/appList/group_id/'+that.$route.params.group_id+'/limit/6/start/'+(page-1)*6,
          // async: false,
          dataType: 'json',
          success: function(res) {
            console.log(res);
            that.lists = res.list;
          },
          error: function(err) {
            console.log(err);
          }
        })
        sessionStorage.applistpage = page;
    },

                //page.vue
    <template>
      <ul class="pagination" v-show="showPagination">
        <li><a @click="change(1)" v-if="showPre"> <!-- &lt;&lt; --> 首页</a></li>
        <li><a  v-if="showPre" @click="prev"><!-- &lt; -->上一页</a></li>
        <li v-for="i in showCount"   @click="change(i)"
          v-bind:class="[currentPage ===   i ? 'active' : '']"
          ><a>{{i}}</a></li>
        <li><a v-if="showTail">...</a></li>
        <li v-bind:class="[currentPage === this.lastPage ? 'active' : '']">
          <a @click="change(this.lastPage)">{{lastPage}}</a></li>
        <li><a v-if="showNext" @click="next"><!-- &gt; -->下一页</a></li>
        <li><a v-if="showNext" @click="change(lastPage)"><!-- &gt;&gt; -->最后一页</a></li>
      </ul>
    </template>
    
    <script>
      module.exports =  {
        components: {
    
        },
        props: {
          total: {
            type: Number,
            require: true
          },
          totalCount:{
            type: Number,
            default:0
          },
          pageSize: {
            type: Number,
            default: 6
          },
          showCount: {
            type: Array,
            default() {
              return [];
            }
          },
          changePage: {
            type: Function,
            default () {
              return function (){};
            }
          },
          apppage: {
            
          },
          applistpage: {
    
          },
          machinepage: {
    
          },
        },
        ready: function(){
          this.currentPage = 1;
          this.lastPage = Math.ceil(this.total / this.pageSize);
          var that = this;
          if(that.apppage) {
            // setTimeout(function(){
              that.change(that.apppage);
            // },100)
          }
          if(that.applistpage) {
            // setTimeout(function(){
              that.change(that.applistpage);
            // },100)
          }
          if(that.machinepage) {
            // setTimeout(function(){
              that.change(that.machinepage);
            // },100)
          }
    
        },
        data:  function() {
          return {
            //当前页数
            currentPage: 1,
            // 最后一页
            lastPage: 1,
            showNext:true,
            showPre:false,
            showTail:false,
            showPagination:true,
          }
        },
        computed: {
          showPagination:function(){
            return this.lastPage == 0?false:true;
          },
          showPre:function(){
            if(this.currentPage == 1){
              return false;
            }else{
              return true;
            }
          },
          showNext:function(){
            if(this.lastPage == 1){
              return false;
            }else{
              if(this.lastPage == this.currentPage){
                return false;
              }
              return true;
            }
          },
          showCount: function(){
                    var $this = this;
                    var ar = [];
                    $this.currentPage = parseInt($this.currentPage);
                    if ($this.currentPage > 3)
                    {
                        ar.push($this.currentPage - 3);
                        ar.push($this.currentPage - 2);
                        ar.push($this.currentPage - 1);
                    }else
                    {
                        for (var i = 1; i < $this.currentPage; i++)
                        {
                            ar.push(i);
                        }
                    }
                    if ($this.currentPage !== $this.lastPage)
                    {
                        ar.push($this.currentPage);
                    }
                    if ( $this.currentPage < ( $this.lastPage - 3 ) )
                    {
                        ar.push($this.currentPage + 1);
                        ar.push($this.currentPage + 2);
                        ar.push($this.currentPage + 3);
                        if ( $this.currentPage < ( $this.lastPage - 4 ) )
                        {
                            $this.$set("showTail", true);
                        }
                    }else
                    {
                        $this.$set("showTail", false);
                        for (var i = ($this.currentPage + 1); i < $this.lastPage; i++)
                        {
                            ar.push(i);
                        }
                    }
                    return ar;
                },
        },
        watch: {
          total (val, oldval) {
            this.lastPage = Math.ceil(this.total / this.pageSize);
            // this.currentPage = 1;
          },
          totalCount(v){
            if(v == 0){
              this.currentPage = 1;
            }
          }
        },
        methods: {
          prev () {
            if(this.currentPage == 1){
              this.showPre = false;
              return;
            }else{
              this.showPre = true;
            }
            this.currentPage = --this.currentPage;
            this.change(this.currentPage);
    
          },
          next () {
            if(this.currentPage == this.lastPage){
              this.showNext = false;
              return;
            }else{
              this.showNext = true;
            }
            this.currentPage = ++this.currentPage;
            this.change(this.currentPage);
    
          },
          change (page) {
            if (page === 1) {
              this.showPre = false;
            }else{
              this.showPre = true;
            }
            if (page === this.lastPage) {
              this.showNext = false;
            }else{
              this.showNext = true;
            }
            if(page< this.lastPage-4){
              this.showTail = true;
            }else{
              this.showTail = false;
            }
            this.currentPage = page;
            this.changePage(page);
          }
        }
      }
    </script>

**8.**活动页主要处理的就是表情的问题了，emoji表情啊，想必大家在发微信墙的时候都想要用表情表达下自己当时的心情吧，但是文字完全不能展现啊～
    好的我们上emoji的了：https://github.com/OneSignal/emoji-picker.git
    当然github中有很多这些可以引用看看，但是试了那么多还是这个比较符合要求，看着代码我还改了那么一丢丢，它原先是禁用了iphone的，我重新把它开上了，但是这个主要是用插件自带的emoji表情，而不是我们的输入法带的，这就非常的不舒服了。经过test，它在iPhone上使用输入法的emoji表情还是支持的，但是在安卓上就不行了，真是，伟大的程序员又要出来解决这些麻烦事了，嘿嘿嘿，虽然到现在还没解决，稍微有点眉目吧，等我再细研究研究这些编码问题先～～
    当然了，如果浏览器或者输入法能像iphone那样支持的话多好啊，方便开发者，方便用户，当然，问题还是要解决的！

好了，这次的博文就到这里了，这里还发现我写的代码注释可能还不是特别的够，虽然语义上够了，但是适当注释还是不要偷懒加上～～又尴尬了～另外md编辑博文其实还是不太方便，要想加多点元素确实还是比较麻烦的，看我这么简洁的写就知道了，可能在稍微有点久的将来我就自己开发一个博文好了，方便自己使用，到时有了再给大家分享把。。！行吧该睡觉了，给各位看博文的说晚安了。。！
