# Web第一次作业记录
## 大致思路：当看到这个作业的时候，感觉手足无措，首先那些给好的样式看不懂，其次是给的数据不懂得怎么调用，再者对js不熟，上课对dom或者事件监听也是云里雾里。所以，整体的思路大概是先把作业要求看懂，有需求才有应用；然后是把任务包里有的东西看清楚，无非就是css样式，data，还有老师上课写的一个参考模板；有了上面这些之后就知道大致要实现的是把页面的元素生成出来，然后给元素添加上样式，给元素添加上需要的事件监听器，以及一些方法的实现；
## 具体思路：
### 1.首先实现对tabs的生成，也就是地域标签，我采用for循环数组用document.elemcreate函数将六个地域选区生成出来并且赋予它们合适的classname（决定样式，也方便获取dom）；
~~~    
    function addLabs () {
      for (var i = 0; i < AREA_DATA.length; i++ ) {
        var node = document.createElement('li');
        var textNode = document.createTextNode(AREA_DATA[i].name);
        node.appendChild(textNode);
        let tabDom = document.getElementById('mytabs');                                          //mytabs是tabs的id
        tabDom.appendChild(node);                                                                //将生成的node节点加入doms树
        tabDom.lastChild.classList.add('tab-item');                                              //每加入一个节点就将其类名定义为'tab-item'
      }
      var firstNode = document.querySelector('li');
      firstNode.classList.add('tab-active');                                                       //默认第一个是加载好的
    }
~~~
### 2.其次是下面album列表的动态生成，用dynamicCreate (a)函数来实现，参数a是代表area的值，由这个选值遍历data里面的所有歌的area，如果符合，那就调用document.elemcreate函数生成其主体（一个div，class=‘album’），以及主体下的个体（一个img作cover，一个img作mask，一个a作名字，一个textelem作innerIndex），难点在于区分各个部分该用什么样的类名标签样式才能符合要求，要点在于掌握queryselector等选择器来选择dom对象进行操作，其中有一个需要注意的地方，那就是通过album来选择其子结点会使程序的可拓展性更强，如果用document来选择对象，那么不仅会比较难，而且当文档流中增添了其他标签，很有可能会选择错位导致bug。
~~~
    function dynamicCreate (a) {
      var dom = document.getElementById('mylist');                                                 //list是包含所有专辑的div容器，里面的album是div，插进图片，名字，作者，年份等
      var j = 0;                                                                                   //变量j用于每添加一个album就自增加一，便于设置每个album的专属id
      for(var i = 0 ; i < ALBUMS_DATA.length ; i++){
        if(ALBUMS_DATA[i].area === a){
          j++;
          var newAlbum = document.createElement('div');                                       //生成album需要的各种子节点
          var coverdom = document.createElement('div');
          var picture = document.createElement('img');
          var maskDom = document.createElement('img');
          var name = document.createElement('a');
          var singer = document.createElement('p');
          var release_time = document.createTextNode(ALBUMS_DATA[i].release_time);
          coverdom.appendChild(picture);
          coverdom.appendChild(maskDom);
          newAlbum.appendChild(coverdom);
          newAlbum.appendChild(name);
          newAlbum.appendChild(singer);
          newAlbum.appendChild(release_time);
          newAlbum.className=("album");
          newAlbum.id=j-1;                                                                    //给每个album一个专属的序号id，便于后续通过事件监听定位到该节点
          dom.appendChild(newAlbum);                                                          //将生成的album节点添加到dom树中
          album = dom.lastChild;                                                              //定位到当前i对应的album，对其内容和属性进行赋值
          var pictureDoms= album.querySelectorAll('img');
          pictureDoms[0].src = ALBUMS_DATA[i].cover;
          pictureDoms[1].classList.add("mask");
          pictureDoms[1].src = "https://y.gtimg.cn/music/photo_new/T002R300x300M000003UybMY27cOsn_1.jpg";
          var divDom = album.querySelector('div');
          divDom.classList.add('cover');
          var nameDom = album.querySelector('a');
          nameDom.innerHTML = ALBUMS_DATA[i].name;
          nameDom.classList.add("title");
          nameDom.href = "#";
          var singerDom = album.querySelector('p');
          singerDom.innerHTML = ALBUMS_DATA[i].singer;
        }
      }
    }
~~~
### 3.对标签的监听，我先实现的是对标签的监听这个较为容易，需要将监听器安装在li标签上，而且这个监听是持续不消失也不变化的，所以每次点击对象，就清空文档流中所有对象，然后动态添加新的dom树结点形成新的页面，细节的地方实现点击之后的tab_active类的变更
~~~
    dynamicCreate(1);                                                                              //默认加载area为1的专辑
    //addLilistener函数是为所有的tab标签添加事件监听，当有click事件时，清空doms树的结点，并动态生成列表以及为列表中的每一项album添加事件监听                                                                      
    function addLilistener () {
      var lis = document.querySelectorAll('li');
      for (var i = 0; i < lis.length; i++){                                                        //事件闭包
        (function (n) {
          lis[i].onclick = function(){
            for(var j = 0; j < lis.length; j++)  lis[j].classList.remove("tab-active");            //出现点击事件，将所有标签都重置属性
            lis[n].classList.add("tab-active");                                                    //为当前点击的dom对象类设置样式
            deleteAll();                                                                           //清空dom树
            dynamicCreate(n+1);                                                                    //n从0开始，area从1开始；动态生成album
            addAlbumlistener();                                                                    //为每一个album增加监听
          }
         }) (i);
      }
    }
~~~
### 4.album对象的监听：实现监听之前需要先实现鼠标放在图片上方显示delete图标，我实现这个的细节在于，我把album的图片和mask两个img插入到了一个position=relative的div里，由于mask的position=absolu，所以mask将会覆盖在album图片上，然后将mask的versible默认值置为0，当mouseon的时候设置为1，就实现了这个功能；然后要实现这么一个功能，鼠标放在mask上，能够将这个mask对应的album删除掉，这个实现需要给每一个album设置一个唯一的id，我是按照1，2，3，4的编号来设置的，与后面为mask添加事件监听的顺序有关，首先给页面中的每一个mask添加事件监听，每一个mask对应一个事件，但是如果不添加id，那么每当删除一个结点，后面的结点便会替代去掉的结点向前填补，再按照原来的mask次序来找dom就会很麻烦甚至不准确，解决办法就是设置id一一对应，后面删除完全按照id来查找而不是按照album在列表中的次序。
~~~
    addLilistener();                                                                               //默认展示第一个，增加监听
    //addAlbumlistener函数用于给每一个album增添一个事件监听，当鼠标点击mask时可以通过mask的序号作为album的id进行定位dom，进行删除操作
    function addAlbumlistener () {
      var masks = document.querySelectorAll('.mask');
      for(var i = 0; i < masks.length; i++){
        (function (n) {
          masks[i].onclick = function () {
            var album = document.getElementById(n);
            for(var j = album.children.length-1; j >=0; j--){
              album.removeChild(album.childNodes[j]);
            }
          album.parentNode.removeChild(album);
          masks[n].onclick = null;                                                              //每删除一个结点就把其对应的事件监控清零，避免内存泄漏
          }
        })(i);
      }
~~~
## 遇到的困难：
### 1.对选择器，dom操作不了解，前期花费了很多时间去看文档和看博客，但是收获很大，看多了并且在自己电脑上写一些小的js语句去实践就能比较快的掌握了；
### 2.对给的css很多参数看不懂，分不清该用给外裹的标签还是给内嵌的图片，最后也是查表看了很多css的参数很多原理才勉强能匹配上完成作业
### 3.删除节点时，一开始是第一个子节点开始往后面删除，但是出现了只能删除一半子节点的问题，原因在于删掉结点之后后面的结点向前移动填补了，再按照原来的次序删除下一个结点实际上是删除下下个结点，解决办法是从后面往前面删除；
### 4.给事件添加监听器时，一开始老是出错，各种各样的小毛病，后面干脆直接写一段小代码先在另外一个文件里测试弄明白了原理再回来实现这个添加
### 5.git的操作不熟练，哪怕有这个比较详细的文档，但是那些分支仓库的概念还是比较模糊，参阅了许多博客的讲解最后才搞明白
