# mescroll.js 快速入门

官网说明： http://www.mescroll.com/

1.下载并引用mescroll插件的css js  
`<link rel="stylesheet" href="https://unpkg.com/mescroll.js@1.4.1/mescroll.min.css">`  
`<script src="https://unpkg.com/mescroll.js@1.4.1/mescroll.min.js" charset="utf-8"></script>`

2. 需要满足下面的布局结构 :  
```
    <div id="mescroll" class="mescroll"> //id可以改,而"mescroll"的class不能删  
        <div> //这个div不能删,否则上拉加载的布局会错位.(可以改成ul或者其他容器标签)  
             //内容...  
         </div>  
    </div>
```

3. 固定mescroll的div高度. 推荐通过定位的方式,简单快捷（当然也可以用其他方法）:  
```
    .mescroll{
        position: fixed;
        top: 44px;
        bottom: 0;
        height: auto; /*如设置bottom:50px,则需height:auto才能生效*/
    }
 ```
    
  4. 创建mescroll对象 :  
  ```
      var mescroll = new MeScroll("mescroll", { //第一个参数"mescroll"对应上面布局结构div的id (1.3.5版本支持传入dom对象)
	       		//如果您的下拉刷新是重置列表数据,那么down完全可以不用配置,具体用法参考第一个基础案例
	       		//解析: down.callback默认调用mescroll.resetUpScroll(),而resetUpScroll会将page.num=1,再触发up.callback
        down: {
            callback: downCallback //下拉刷新的回调,别写成downCallback(),多了括号就自动执行方法了
        },
        up: {
            callback: upCallback, //上拉加载的回调
            //以下是一些常用的配置,当然不写也可以的.
            page: {
                num: 0, //当前页 默认0,回调之前会加1; 即callback(page)会从1开始
                size: 10 //每页数据条数,默认10
            },
            htmlNodata: '<p class="upwarp-nodata">-- END --</p>',
            noMoreSize: 5, //如果列表已无数据,可设置列表的总数量要大于5才显示无更多数据;
                    //避免列表数据过少(比如只有一条数据),显示无更多数据会不好看
                    //这就是为什么无更多数据有时候不显示的原因.
            toTop: {
                //回到顶部按钮
                src: "../img/mescroll-totop.png", //图片路径,默认null,支持网络图
                offset: 1000 //列表滚动1000px才显示回到顶部按钮	
            },
            empty: {
                //列表第一页无任何数据时,显示的空提示布局; 需配置warpId才显示
                warpId:	"xxid", //父布局的id (1.3.5版本支持传入dom元素)
                icon: "../img/mescroll-empty.png", //图标,默认null,支持网络图
                tip: "暂无相关数据~" //提示
            },
            lazyLoad: {
                    use: true, // 是否开启懒加载,默认false
                    attr: 'imgurl' // 标签中网络图的属性名 : <img imgurl='网络图  src='占位图''/>
                }
        }
      });
  ```
      
5. 处理回调 :  
```
   //上拉加载的回调 page = {num:1, size:10}; num:当前页 默认从1开始, size:每页数据条数,默认10
    function upCallback(page) {
        var pageNum = page.num; // 页码, 默认从1开始 如何修改从0开始 ?
        var pageSize = page.size; // 页长, 默认每页10条
        $.ajax({
            url: 'xxxxxx?num=' + pageNum + "&size=" + pageSize,
            success: function (data) {
                var curPageData = data.xxx; // 接口返回的当前页数据列表
                var totalPage = data.xxx; // 接口返回的总页数 (比如列表有26个数据,每页10条,共3页; 则totalPage值为3)
                var totalSize = data.xxx; // 接口返回的总数据量(比如列表有26个数据,每页10条,共3页; 则totalSize值为26)
                var hasNext = data.xxx; // 接口返回的是否有下一页 (true/false)

                //联网成功的回调,隐藏下拉刷新和上拉加载的状态;
                //mescroll会根据传的参数,自动判断列表如果无任何数据,则提示空,显示empty配置的内容;
                //列表如果无下一页数据,则提示无更多数据,(注意noMoreSize的配置)

                //方法一(推荐): 后台接口有返回列表的总页数 totalPage
                //必传参数(当前页的数据个数, 总页数)
                //mescroll.endByPage(curPageData.length, totalPage);

                //方法二(推荐): 后台接口有返回列表的总数据量 totalSize
                //必传参数(当前页的数据个数, 总数据量)
                //mescroll.endBySize(curPageData.length, totalSize);

                //方法三(推荐): 您有其他方式知道是否有下一页 hasNext
                //必传参数(当前页的数据个数, 是否有下一页true/false)
                //mescroll.endSuccess(curPageData.length, hasNext);

                //方法四 (不推荐),会存在一个小问题:比如列表共有20条数据,每页加载10条,共2页.
                //如果只根据当前页的数据个数判断,则需翻到第三页才会知道无更多数据
                //如果传了hasNext,则翻到第二页即可显示无更多数据.
                //mescroll.endSuccess(curPageData.length);

                //curPageData.length必传的原因:
                1. 使配置的noMoreSize 和 empty生效
                2. 判断是否有下一页的首要依据:
                当传的值小于page.size时(说明不满页了), 则一定会认为无更多数据;
                比传入的totalPage, totalSize, hasNext具有更高的判断优先级;
                3. 当传的值等于page.size时, 才会取totalPage, totalSize, hasNext判断是否有下一页
                传totalPage, totalSize, hasNext目的是避免方法四描述的小问题

                //设置列表数据
                //setListData(curPageData);//自行实现 TODO
            },
            error: function (e) {
                //联网失败的回调,隐藏下拉刷新和上拉加载的状态
                mescroll.endErr();
            }
        });
    }
```
