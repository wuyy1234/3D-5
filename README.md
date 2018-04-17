# 3D-5  
## 编写一个简单的鼠标打飞碟（Hit UFO）游戏  
> * 编写一个简单的鼠标打飞碟（Hit UFO）游戏  
> * 游戏内容要求：  
>   * 游戏有 n 个 round，每个 round 都包括10 次 trial；  
>   * 每个 trial 的飞碟的色彩、大小、发射位置、速度、角度、同时出现的个数都可能不同。它们由该 round 的 ruler 控制；  
>   * 每个 trial 的飞碟有随机性，总体难度随 round 上升；  
>   * 鼠标点中得分，得分规则按色彩、大小、速度不同计算，规则可自由设定。  
> * 游戏的要求：  
>   * 使用带缓存的工厂模式管理不同飞碟的生产与回收，该工厂必须是场景单实例的！具体实现见参考资源 Singleton 模板类  
>   * 尽可能使用前面 MVC 结构实现人机交互与游戏模型分离  

### 整体解决方案

#### MVC架构

> * 游戏总体结构：
>   * 使用MVC架构，用一个Controller控制各个下属组件，各个下属组件要相互联系的时候只能先调用Controller里面的接口。  
```  
//示范接口结构和Contoller与各个下属分支，下属分支之间的相互联系：  
//例如View场景要“删除”某个飞盘（其实是把它移出摄像机视觉外）
//由于是MVC架构，不能自己直接对飞盘进行操作
//解决方法：
//调用Contoller的public void destroyDisk(GameObject disk)函数，
//再通过这个函数分别调用View和Factory里面的destoryDisk来从视觉和逻辑上将其“删除”

//View里面的：
private void Update(){
    if (usingDisks[i].transform.position.y <=2){
        GameScenceController.getGSController().destroyDisk(usingDisks[i]);
        GameScenceController.getGSController().subScore();
    }
}
//这里的usingDisks[i].transform.position.y <=2判断Disk已经在视野之外如果扔未被点击，
//需要将其“摧毁”，于是调用下面在Controller的接口
//Controller里面的：
public class Controller{
    public void destroyDisk(GameObject disk)
        {
            _scence.destroyDisk(disk);
            _diskFactory.recycleDisk(disk);
        }
}
//这时Controller要在视觉和逻辑上隐藏飞盘，
//所以具体的操作又交回给负责视觉上的View和负责对盘子进行逻辑操作的DiskFactory
//View里面的destroyDisk(disk)
public void destroyDisk(GameObject disk)
    {
        disk.GetComponent<Rigidbody>().Sleep();
        disk.GetComponent<Rigidbody>().useGravity = false;
        disk.transform.position = new Vector3(0f, -99f, 0f);//将其隐藏到摄像机外
    }
//Factory里面的Destory，要留意的是因为还考虑到了工厂模式，
//所以这里的Destory其实是将Disk放回待用序列uselessDisks等待被下一次调用。
public void recycleDisk(GameObject disk)
    {
        int index = usingDisks.FindIndex(x => x == disk);//找到usingDisks里面要回收的disks放回uselessDisks
        uselessDisks.Add(disk);
        usingDisks.RemoveAt(index);
    }
```  

#### 工厂模式  
>  *   因为如果代码频繁对游戏对象Instantiate和Destory会影响游戏性能
>  *   所以当一个物体不用的时候将其存进一个存储序列，等待下次要重新调用的时候再将其取出放入使用序列
>  *   下面以飞盘工厂为示例：
``` 

```

>  *   
>  *   
>  *   
>  *   