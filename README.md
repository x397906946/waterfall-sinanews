<!DOCTYPE html>
<html>
<head>
<script src="//code.jquery.com/jquery-1.9.1.min.js"></script>
 <meta charset="utf-8">
<meta http-equiv="X-UA-Compatible" content="IE=edge,chrome=1">
<title>waterfull-lazyload-ajax demo</title>
<meta name="description" content="">
<meta name="keywords" content="">
<link href="" rel="stylesheet">
  <style>
    * {
      margin: 0;
      padding: 0;
      box-sizing: border-box;
    }
    ul,li {
      list-style: none;
    }
    
    .clearfix:after {
      content: '';
      display: block;
      clear: both;
    }
    #pic-ct {
      position: relative;
    }
    #pic-ct .item {
      position: absolute;
      padding: 0 0 10px 0;
      width: 280px;
      margin: 10px;
      border: 1px solid #dfdfdf;
      background: #fff;
      transition: all .8s;
    }
    #pic-ct .item img {
      margin: 10px;
      width: 260px;
    }
    #pic-ct .item .header {
      height: 25px;
      margin: 0 12px;
      border-bottom: 1px solid #dbdbdb;
    }
    #pic-ct .item .desp {
      font-size: 12px;
      line-height: 1.8;
      margin: 10px 15px 0;
      color: #777371;
    }
    .hide {
      display: none;
    }
  </style>
</head>
<body>
  <div class="ct-waterfall">
    <ul id="pic-ct" class="ct clearfix">

      <li class="item hide"></li>
    </ul>
    <div id="load"></div>
  </div>
  <script>
    /* 整体思路: */
    // 1.获取数据
    // 2.将数据变为DOM结构,以瀑布流的方式部署到页面上
    // 3.当页面滚动至底部再次获取数据循环

    var curPage = 1, //当前页面
        perPageCount = 9, //获取数据个数
        interval = 100, //定时器间隔
        clock, //定时器
        itemArr, //瀑布流 定义空数组,存放每一列的总高度
        colNum //瀑布流个数

    reset()
    start()

    // 获取数据
    function getData(callback){
      $.ajax({
        url: 'http://platform.sina.com.cn/slide/album_tech',
        dataType: 'jsonp',
        jsonp: 'jsoncallback',
        data:{
          app_key: '1271687855',
          num: perPageCount, 
          page: curPage
        }
      }).done(function(ret){
        if(ret && ret.status && ret.status.code === "0"){ //判断数据是否正常
          callback(ret.data) //回调返回的数据
          curPage++
        }else{
          console.log('error')
        }
      })
    }

    //生成DOM节点
    function getNode(item){
      console.log(item)
      var tpl = ''
		  tpl += '<li class="item">'
	  	tpl += ' <a href="'+ item.url +'" class="link"><img src="' + item.img_url + '" alt=""></a>'
		  tpl += ' <h4 class="header">'+ item.short_name +'</h4>'
		  tpl += '<p class="desp">'+item.name+'</p>'
      tpl += '</li>'
      return $(tpl)
    }

    //初始化瀑布流数组
    function reset(){
      itemArr = [],
      //由父容器宽度与item宽度之比得到瀑布流个数
      colNum = Math.floor($('#pic-ct').width()/$('.item').outerWidth(true))
      //初始化瀑布流高度为0
      for(var i=0; i<colNum; i++){
        itemArr.push(0)
      }
    }

    //瀑布流布局
    function WaterFall($node){
      var minHeight = Math.min.apply(null, itemArr),
          minIdx = itemArr.indexOf(minHeight)
      $node.css({
        left: $('.item').outerWidth(true) * minIdx,
        top: minHeight
      })
      itemArr[minIdx] += $node.outerHeight(true)
      //需要将父容器ul高度设置成数组中的最大高度,因为数组元素item都是绝对定位,脱离了文档流,父容器高度坍塌,需要设置高度
      $('#pic-ct').height(Math.max.apply(null, itemArr))
    }

    //实现懒加载
    function isVisible($el){
      //判定条件：item的高度 小于 窗口高度与窗口滚动距离之和
      //还没有滚动至目标就开始加载数据，优化体验
      if($el.offset().top < $(window).height() + $(window).scrollTop() + 500){
        return true
      }else{
        return false
      }
    }
    //页面滚动至底部时，继续加载数据
    $(window).on('scroll', function(){
      if(clock){
        clearTimeout(clock);
      }
      clock = setTimeout(function(){
        if( isVisible($('#load')) ){
          start()
        }
      }, interval);
    })

    //思路实现:获取数据,变成DOM结构,瀑布流方式部署到页面
    function start(){
      getData(function(newList){
        $(newList).each(function(idx, item){
          //将数据变为DOM结构
          var $node = getNode(item)

          //待图片加载完毕后，将节点放在页面上，并以瀑布流部署
          $node.find('img').on('load', function(){
            $('#pic-ct').append($node)
            WaterFall($node)
          })
        })
      })
    }

    //窗口大小变化时，瀑布流布局随之改变
    $(window).on('resize', function(){
      reset()
      $('.item').each(function(){
        WaterFall($(this))
      })
    })
  </script>
</body>>
