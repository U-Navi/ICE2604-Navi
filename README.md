[TOC]

## 传送门
[https://github.com/huskydoge/SJTU-Classroom](https://github.com/huskydoge/SJTU-Classroom)
<br>
<!-- 芝士雪豹 -->
## 代码运行操作指南

推荐使用的IDE为Webstorm和Pycharm（我们网站的主要代码是在这两个IDE上写的）。我们网站代码的语法基本是vue2，但是当初创建项目的时候用的是vue3（Vue3基本向下兼容Vue2，除了一些细节）。对于缺失的各种库，请自行安装。**由于经验不足，我们没有采用按需引入库的方式，导致文件容量较为庞大，这需要改进**。

### **网站启动流程**

1. 用webstorm打开代码文件夹内的ice-001项目，运  `npm-serve`。

2. 进入 src/server 文件夹, 在终端中输入`node app.js`的文件地址和`node avatar.js`的文件地址以启动**主接口和头像接口**。

   ![截屏2022-12-27 15.59.22](https://tva1.sinaimg.cn/large/008vxvgGly1h9ifd8vsynj30ne0jkt9j.jpg)

3. 用pycharm打开classroomAPI项目文件夹，运行 `main.py`以启**动词云图接口和搜索接口**。


![截屏2022-12-27 16.29.52](http://rnjlk4ikb.hd-bkt.clouddn.com/md/2022-12-27-094247.png)

4. 启动文件夹内的 `elasticsearch+kibana`, 使**搜索引擎**正常运行。<u>**注意！由于es的数据是储存在本地的，请使用文件夹内的es程序！**</u>

### **动态数据爬取**

* 为了刷新教室折线图界面的数据，请运行`Crawlers/dataCrawler/everyHourData/everyHourCrawler.py`

* 为了刷新主页的实时动态数据，请运行`Crawlers/dataCrawler/liveData/liveDataCrawler.py`

<u>*至此网站应该已经可以正常运行。如果出现问题，请邮箱联系`hbh001098hbh@sjtu.edu.cn`。*</u>

### **注意⚠️**

1. 由于搜索引擎中的数据是通过logstash手动上传的，并没有使用自动化脚本，所以数据更新后需要重新用logstash运行

`/logstash-7.5.2/config`文件夹中的 [lost.conf](ICE Code/logstash-7.5.2/config/lost.conf)  ｜[found.conf](ICE Code/logstash-7.5.2/config/found.conf)  ｜[buildingInfo.conf](ICE Code/logstash-7.5.2/config/buildingInfo.conf) 三个文件。具体操作方法可以参考这里：[sql数据导入es](https://blog.csdn.net/ShuSheng0007/article/details/123021600)。

2. 如果找不到对应路径，推荐先用您电脑上的文件系统对本项目内文件进行搜索🔍。<u>由于项目文件多次更新，很有可能出现文件路径与报告中不匹配的情况，请谅解。</u>

*****

## 技术亮点

### **多级评论区的实现**

从网站的设计过程来看，评论区的设计较为复杂。我们的评论区支持用户点赞、支持用户删除自身评论也支持多层级评论。

这些功能的实现首先建立在我们为每个用户都**分配了userId**的基础之上。

![截屏2022-12-27 15.39.50](https://tva1.sinaimg.cn/large/008vxvgGly1h9iesvemitj311a0443z7.jpg)

有了用户的id，我们可以将一条一级评论的属性与用户的id相联系。

比如，我们用下图（储存一级评论的数据表）中的favor字段记录为该评论点赞的用户。并与前端配合实现了：当且仅当favor字段中不存在某个用户的id，它才能点赞。

![截屏2022-12-27 15.44.18](https://tva1.sinaimg.cn/large/008vxvgGly1h9iexixwxhj31lu0jk79z.jpg)

用户删除评论的实现机理和点赞十分类似，便不再赘述。

而比较困难的是多层评论的实现。如何设计数据库结构成为一大问题。最终我们的选择是，每个一级评论的子评论字段里用json的格式存储各个子级评论。举例来看：

```json
{
	"replyList": [{
		"date": "2022年12月01日 23:21:46 星期四",
		"favour": [79, 118],
		"rootId": 61,
		"userId": 79,
		"content": "在做了在做了，别催，催也没用，急死你",
		"username": "Dr. Miao",
		"avatarUrl": "https://tse2-mm.cn.bing.net/th/id/OIP-C.UsavMQ7MV08TtUiCaEKTAAAAAA?w=198&h=198&c=7&r=0&o=5&dpr=2&pid=1.7",
		"replyName": "Boss"
	}, {
		"date": "2022年12月20日 14:14:48 星期二",
		"favour": [118],
		"rootId": 61,
		"userId": 106,
		"content": "做完啦",
		"username": "root",
		"avatarUrl": "https://ts1.cn.mm.bing.net/th/id/R-C.2f0bf39a1525331307fa77da4104c50b?rik=IPiRE327Dtghpg&riu=http%3a%2f%2ftva3.sinaimg.cn%2flarge%2f006yt1Omgy1grcdi199vbj30u00tmnpd.jpg&ehk=hsqxxbCktnIp%2fckZr9whtjyyUyQmN4tqXZMCawc23bI%3d&risl=&pid=ImgRaw&r=0",
		"replyName": "Dr. Miao"
	}, {
		"date": "2022年12月20日 20:43:02 星期二",
		"favour": "[]",
		"rootId": 61,
		"userId": 118,
		"content": "我们是冠军！",
		"username": "visitor",
		"avatarUrl": "http://localhost:8848/6f18a48120ae3b85036864fc40743723",
		"replyName": "Dr. Miao"
	}, {
		"date": "2022年12月20日 20:43:19 星期二",
		"favour": "[]",
		"rootId": 61,
		"userId": 118,
		"content": "我们是冠军！我们是冠军！我们是冠军！我们是冠军！我们是冠军！我们是冠军！我们是冠军！我们是冠军！我们是冠军！我们是冠军！我们是冠军！我们是冠军！我们是冠军！我们是冠军！我们是冠军！我们是冠军！我们是冠军！我们是冠军！我们是冠军！我们是冠军！我们是冠军！我们是冠军！我们是冠军！我们是冠军！我们是冠军！",
		"username": "visitor",
		"avatarUrl": "http://localhost:8848/6f18a48120ae3b85036864fc40743723",
		"replyName": "visitor"
	}]
}
```

从以上内容中可以看到，子级评论和在一级评论的基础之上，还添加了一个`rootId`用来标识其父级评论。这样清晰的数据结构设计让前端对多级评论功能的实现更为便利。

### **如何实现多层菜单+多标签页的页面呈现**

一般而言，用`Flask`做的网站在实现不同页面的呈现时通常采取的策略是通过`url`的跳转而呈现不同的`html`文件。而本网站用的框架为`Vue`, 对于不同页面的呈现，我们综合使用了以下两种策略：

* 加入`v-router`。同样是通过`url`跳转，不同的是`Vue`可以只改变网站中某一局部的呈现内容，也即改变这部分所展示的组件。

  比如，网站中侧边栏的跳转用的便是router的切换。下面是我们router文件的一部分内容。

  ```javascript
  // router文件
  //import all Components Needed
  const routes = [
      {
          path: '/home',
          name: 'home',
          //路由嵌套
          children:[
              {
                  path: 'dynamic',
                  name: 'dynamic',
                  component: mainPage,
                  //路由嵌套
              },
              {
                  path: 'intro',
                  name: 'intro',
                  component: introPart,
              },
              {
                  path: 'dongshang',
                  name: 'dongshang',
                  component: dongShang,
              },
  					........
          ]
      },
  ]
  
  const router = createRouter({
      mode: 'history',
      history: createWebHashHistory(),
      routes
  })
  
  export default router;
  ```

* 用逻辑变量+`v-if`。通过改变`v-if="flag"` 和`flag` 变量随用户点击行为而在`true`和`false`间转换的方法实现组件的显示和隐藏。

以上两种方式在我们的网站中都有所体现：

![截屏2022-12-27 19.06.46](http://rnjlk4ikb.hd-bkt.clouddn.com/md/2022-12-27-111050.png)

![截屏2022-12-27 19.07.00](http://rnjlk4ikb.hd-bkt.clouddn.com/md/2022-12-27-111326.png)

### **接口设计**

考虑到不同数据的特殊性，我们在网站设计中综合使用了`post`和`get`两种数据请求方式。

比如，对于一般的教室数据等，我们使用了`get`请求；而对于用户的信息，比如用户登陆时密码是否匹配的过程，我们全程使用`post`请求，以在我们力所能及的范围内保证用户的密码不被泄露。

另一方面，我们还对返回的数据的合适类型进行了思考。当用户输入密码准备登陆时，暴露在外的接口应该仅仅返回不同的`status`表征输入的用户名和密码是否匹配或者该账号是否有效，而不应该返回用户储存在数据库中的密码，在前端进行判定。

我们给出两种代码：

```javascript
// 登陆账户，在后端进行判断
exports.signIn = (req, res) => {
    const sql = "SELECT * FROM Users WHERE username = ?";
    db.query(sql, [req.body.name],(err, data) => {
        if(err) {
            return res.send('错误：' + err.message)
        }
        // 如果返回长度为0，说明没有注册！
        if(data.length === 0){
            data = {status: 2}
        }
        else if(data[0].password === req.body.password){
            data = {status: 1};
        } else {
            data = {status: 0};
        }
        res.send(data)
    });
}

// 获得账户密码, 返回前端匹配，安全性堪忧
exports.getPassword = (req, res) => {
    const sql = "SELECT * from Users WHERE username = ?";
    db.query(sql, [req.query.name],(err, data) => {
        if(err) {
            return res.send('错误：' + err.message)
        }
        res.send(data[0])
    });
}
```

<u>在项目的代码文件中，我们同时保存了这两种写法，但是仅用了安全的那种交互方式，并给出了注释。</u>

### **可视化中echarts图表元素点击跳转的设计**

在我们的理念里，图表不应该仅仅是数据的载体，更应承担着用户交互以及界面过渡的功能。

**在网站构思初期，我们就提出了希望用气泡图的方式，以气泡的大小直观地表征数据的大小。但是，这样的展示方式虽然有了直观的美感，但是数据仍然是单薄的，交互是低效的。从而我们有了点击图形元素跳转到教室界面的构想。**

`echarts`是一个十分强大的库，它大大增加了可视化的精彩程度。而为了实现上述构想，我们通过浏览了echarts的官方文档和官方的示例代码，找到了`myChart.on`函数，成功地实现了我们的构想。

```vue
myChart.on('click',function (params){
		setClass(params.data.name)
     })
})
```

这一部分的代码是实现点击下图中的圆圈部分跳转到相应的教室的核心部分。

其中`setClass(params.data.name)` 函数将点击元素返回的相应元素名，也即教室名传入相应组件的对外数据接口(`props`)，并改变相应的与`v-if`关联的逻辑变量，从而实现跳转到相应教室界面的功能。![截屏2022-12-27 20.42.12](http://rnjlk4ikb.hd-bkt.clouddn.com/md/2022-12-27-124218.png)

当然，在这一跳转的过程中，<u>还出现了返回气泡图界面后图消失，点击元素不跳转等一系列问题。</u>经过我们的探索，我们发现问题的根源在于对`Vue`的生命周期浅显理解。为了达到理想效果，我们需要合理使用一系列`Vue`的函数，比如`mounted`、`created`和`watch`等。这一过程中，我们深深体会到了`Vue`框架上手容易、深入难的一面。具体内容便不在这里展开叙述。
