#MUI
  [MUI新手指南](http://dev.dcloud.net.cn/mui/ui/)
   >mui.js  
   
   >(1)封装了基本的事件绑定，选择器操作等(类似jQuery)。  
   
   >(2)对移动端事件进行了封装(tap.swipe)。  
   
   >(3)封装了WebView的slide事件。  
   
#H5+
  [H5+Specification API](http://www.html5plus.org/doc/h5p.html)
   > (1)配合Native.js实现对原生端功能的调用(比较稳定，也比较易用)。  
   
   > (2)提供了WebView组件（三种集成方式）  
   
#遇到的问题和解决方案
##(1)父子WebViewback事件处理说明
> 如果父WebView重写了mui.back()方法，那么子WebView重写mui.back()方法无效，返回键触发后，默认行为：首先关闭子WebView，然后触发父WebView的mui.back()方法。如果需要在子WebView页面屏蔽返回键的事件处理，则需要在子WebView页面，进行如下初始化操作：    

>     mui.init({  
>     
>                keyEventBind: {  
>                
>                    backbutton: false  
>                    
>                }  
>                
>      });  
>      
>     设置成功后，子WebView在接收到back事件后，不会关闭页面，然后back事件分发，父级WebView的mui.back()触发。  
     
     
   > 父级WebView和子WebView的事件分发问题，比如返回键问题，管理中心的审核详情退出后列表页面消失问题
          mui的父级和子级WebView的事件传递遵循Android api的父子控件的事件传递机制，遵循自上而下的事件分发机制。即父级WebView接收到系统事件若拦截则自己处理，不拦截则分发给子WebView处             理，子WebView处理后不选择消费掉此事件即返回false则继续由父级WebView处理。
          
##(2)iOS版本中使用plus.storage组件
  >    iOS版本中，storage组件的setItem方法无法存储value为整型类型的值，如果需要保存，则需要转化为字符串类型处理。如下所示：
      plus.storage.setItem(key,value+'');
      Android版本中不存在此问题。

##(3)mui.back实现为window.history.back();与Native端返回键的处理则有H5+ 部分来处理

##(4)由于是打包在本地运行，不存在Ajax的跨域问题。

##(5)Angular 
   > \$scope数据绑定采用  \$scope.\$applyAsync执行。(原因在于controller在初始化的过程中会对当前DOM中的表达式进行绑定，如果当前DOM绑定并没有完成，\$scope的直接赋值行为会与未绑定完成的过程冲突，导致绑定失败，而\$scope.\$applyAsync绑定，会通知事件处理器，随后需要绑定数据，类似于排队)。

##(6)plus.storage组件提供了全局存储方案，可以实现缓存(页面事件缓存)、全局变量存储(状态绑定)等场景。

##(7)页面参数传递
   > (1)如果是同一WebView之间的页面跳转，则使用QueryString机制传递，目标页面要实现QueryString参数处理方法。  
   
 >    (2)如果是不同WebView之间参数传递，有两种实现方式，一种是与(1)相同，一种是使用WebView的extras参数，在目标页面可以直接引用(plus.webview.currentWebView.key)。  
    
  >  (3)大数据量传递，则可以用plus.storage.  
    

##(8)下拉刷新列表的实现，为列表和数据绑定都在同一个页面(避免处理返回键的问题)。

##(9)下拉列表事件触发
  >   默认情况下，如果当前列表的数据项，不足一页(亦即数据无法占据屏幕整个高度)，则不会响应列表的事件，此时应该隐藏底部的“加载更多”。因此实现的方案为：  
  
 > $.each(document.querySelectorAll('.mui-slider-group .mui-scroll'), function(index, pullRefreshEl) {  

> var loading = document.querySelectorAll('.mui-slider-group .mui-scroll')[index];  
 
> $(pullRefreshEl).pullToRefresh({  

> down: {  

> callback: function() {  
> 
>                          var self = this;  
>                          
>                          console.log("下拉事件触发");  
>                          
>                          var currentLoading = loading;  
>                          
>                           //标记下拉事件  
>                           
>                          window.PFE = 1;  
>                          
>                          refreshData(index);  
>                          
>                          //通知组件停止下拉刷新操作
>                          self.endPullDownToRefresh();  
>                          
>                          if (jQuery(jQuery(currentLoading).children("ul").get(0)).height() <    
>                          
>                          jQuery(currentLoading).parent().parent().height()) {  
>                            
>                          //计算当前显示区域高度，如果不足一页，则不用显示底部的“加载更多”  
>                          
>                          self.endPullUpToRefresh(true);  
>                          
>                          } else {  
>                          
>                                     self.endPullUpToRefresh();  
>                                     
>                                    }  
>                                    
>                          }  
>                         
>                            },  
>                            
>                            up: {  
>                            
>                                auto: true, //auto:true  
>                                自动执行下拉事件一次，页面加载，自动执行，可以执行：计算当前显示区域高度，如果不足一页，则不 >  
>                                用显示底部的“加载更多”。
>                                  
>                                callback: function() {  
>                                
>                                    console.log("上拉事件触发");  
>                                    
>                                    var self = this;  
>                                    
>                                   var currentLoading = loading;  
>                                    
>                                    //标记上拉事件  
>                                    
>                                    window.PFE = 2;  
>                                    
>                                    refreshData(index);  
>                                    
>                                    //判断首次加载的数据是否超过当前显示区域  
>                                    
>                                    if (jQuery(jQuery(currentLoading).children("ul").get(0)).height() <  
>                                    jQuery(currentLoading).parent().parent().height()) {  
>                                    
>                                        console.log("HJKKLLKL:<L:<L");  
>                                        
>                                        self.endPullUpToRefresh(true);  
>                                        
>                                    } else {  
>                                    
>                                        self.endPullUpToRefresh(false);  
>                                        
>                                    }  
>                                    
>                                }  
>                                
>                            }  
>                            
>                        });  
>                        
>                    });  
>                    

  ##(10)手势事件  
  
> mui.init({  

> gestureConfig:{  
>
> tap:true,//默认为true  
>
> doubletap:true,//默认为false  
>
> longtap:true,//默认为false  
>
> swipe:true,//默认为true  
>
> drag:true,//默认为true  
>
> hold:false,//默认为false，不监听  
>
> release:false//默认为false，不监听  
>
> }});

## (11)建议使用jQuery
 [jQuery API](http://api.jquery.com/)
> (1)事件绑定更为方便。  

> (2)Ajax可控参数也较多。  


## (12)Angular 数据绑定
[Angular API](https://docs.angularjs.org/api)
> View层：添加一个隐藏的button，调用angular.element("#updateNoticeCount").trigger('click');的形式来触发数据加载和绑定。

## (13)Angular和DOM间的参数传递
> 除了页面的HTML元素之外。可以使用window对象(Angular的controller会通过DI，注入$window对象,亦即DOM中的window对象)。比如需要分页的列表加载场景。

##  (14)实现按压引起元素样式变更，可以使用如下方案(主要是hold和release事件)：  

> mui('.mui-bar-tab').on('hold', 'a', function(e) {  
>
>                //按住屏幕  
>                
>                jQuery(this).removeClass('mui-active');  
>                
>                jQuery(this).addClass('mui-active');  
>                
>            });  
>            
>            mui('.mui-bar-tab').on('release', 'a', function(e) {  
>            
>                //离开屏幕  
>                
>                jQuery(this).removeClass('mui-active');  
>                
>            });   
            

## (15)回退刷新场景
> 回退刷新场景：比如说从详情页返回列表页的场景中：可以利用mui.init({beforeback:function(){}}),即需要在返回键触发时完成一些操作(rerurn true:则继续执行返回键处理流程，return false:则终止返             回键处理流程)，再此回调中可以实现刷新上一个WebView数据的功能。

## (16)HTML页面中a标签的href属性处理
 > 如果是#，则tap事件中的event对象，并不能阻止冒泡。改为javascript:void(0)或者别的其他啥都行，就可以调用event对象的preventDefault或者stopPropagation阻止默认行为。

## (17) mui.fire()
> 如果要触发另一个 WebView的事件，则使用mui.fire().典型场景为：侧滑功能的实现，涉及侧滑页和主页面。

## (18)图片不存在的布局异常
> 针对页面中存在图片展示的场景，如果当前图片链接地址无效(或者说图片不存在)，MUI默认的样式会撑破页面布局，导致整体布局异常。解决方案有两个：  
>
>                1.利用js的image对象的onerror事件在检测到事件触发时，则说明图片地址无效，此时可以删除DOM中的该img元素。  
>                
>                2.给img元素设置默认的大小  
                

## (19)plus.nativeUI.showWaiting();
> 会使页面所有事件都不触发。(可以利用这个特性，让页面无法操作，必须等待执行完成，执行关闭plus.nativeUI.closeWaiting();)

## (20)backbutton
> 设置backbutton为false,表示当前WebView不处理返回键，表现的结果为：当前WebView对返回键不响应，只是在整个返回键的事件分发流程中忽略当前WebView，返回键 保持原有事件传递机制。  

> mui.init({  
>
>            keyEventBind: {  
>            
>                backbutton: false  
>                
>            }  
>            
>  });  
 

## (21)plus.runtime.openURL()可以利用本机的浏览器打开URL。

## (22)plus.runtime.launchApplication()可以打开本机安装的APP。

## (23)调查问卷单选、多选、问答题等js的数据组织和接口处理问题
> 用js模拟java中bean和ArrayList的处理数据的形式存储单选、多选、问答题的答案，用"|"分割问题类型，用","分割选项，最后变成字符串传输，服务端按照既定形式分割后存储。   

> Tips:使用JavaScript实现面向对象  

> (1)面向对象([Immutable.js](https://facebook.github.io/immutable-js/))  
 
> (2)函数式编程([jQuery](http://api.jquery.com/))  
     
## (24)选项卡切换与页面联动及数据加载问题
> 选项卡的href="#"对应模块的div的id，不能有空格。
数据加载时mui.plusReady之后，触发Angular JS的controller加载数据，需要兼顾缓存更新，下拉和上拉加载数据。

## (25)mui上拉加载分页不响应问题
> mui默认情况，列表数据超过屏幕时才能响应上拉加载的手势，需要修改mui-pull的样式

## (26)广告切换效果不正确问题
> 除了用Angular Js的ng-repeat加载需要的广告数据之外，在mui-slider-group mui-slider-loop标签之内需要手动添加广告的第一项和最后一项，即广告[list.length-1]--广告数据--广告[0]

## (27)无限列表加载问题
> 如首页的关于商会，项目推广等
> 接口在查询列表时需要递归查出下一级列表，返回给客户端，客户端决定加载的页面，直到没有下一级列表为止

## (28)上拉加载第一页完毕且没有下页数据后，上拉标签显示没有更多数据问题
> 需要自动加载两页才能显示出该提示

## (29)通知、公告、调查问卷、投票状态更新问题
> 通知公告各栏目需要更新未读、已读、已提交状态，其中通知栏目需要更新是否和不确定三种状态，不能混淆 
