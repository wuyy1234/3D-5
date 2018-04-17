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

* 视频连接：http://v.youku.com/v_show/id_XMzU0NTc5MjUxMg==.html?spm=a2h3j.8428770.3416059.1

#### 整体UML化简图
   
   <img src="http://imglf5.nosdn.127.net/img/Z281REhERnhNZlhsYXVvUTNMV2JZMkZHYWdiMFk3Z2VLcHg5SVhyNVNRNVgrRGY0WUYrMFZ3PT0.png?imageView&thumbnail=500x0&quality=96&stripmeta=0"  /> 

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
for (int i = 0; i < usingUFOs.Count; i++) {
			if (usingUFOs [i].transform.position.y < -10) {//判断出界,回收
				//model.getModel().subScore(10);
				Controller.getController().deleteUFO(usingUFOs[i]);

			}
		}

//这里的usingDisks[i].transform.position.y <=2判断Disk已经在视野之外如果扔未被点击，
//需要将其“摧毁”，于是调用下面在Controller的接口
//Controller里面的：
public class Controller{
   public void deleteUFO(GameObject UFO){
		UFOFactory.getFactory().deleteUFO(UFO);
		Scene.getScene().deleteUFO(UFO);
	}

}
//这时Controller要在视觉和逻辑上隐藏飞盘，
//所以具体的操作又交回给负责视觉上的View和负责对盘子进行逻辑操作的DiskFactory
//View里面的destroyDisk(disk)
public void  deleteUFO(GameObject UFO){
		UFO.GetComponent<Rigidbody> ().Sleep ();
		UFO.GetComponent<Rigidbody> ().useGravity = false;
		UFO.transform.position = new Vector3 (0f, 20f, 0f);

	}
//Factory里面的Destory，要留意的是因为还考虑到了工厂模式，
//所以这里的Destory其实是将Disk放回待用序列uselessDisks等待被下一次调用。
public void deleteUFO(GameObject UFO)
		{
			//Debug.Log ("deleteUFO");
			int index = usingUFOs.FindIndex(x => x == UFO);//找到usingUFOs里面要回收的UFOs放回uselessUFOs
			uselessUFOs.Add(UFO);
			usingUFOs.RemoveAt(index);
		}
```  

#### 工厂模式  
>  *   因为如果代码频繁对游戏对象Instantiate和Destory会影响游戏性能
>  *   所以当一个物体不用的时候将其存进一个存储序列，等待下次要重新调用的时候再将其取出放入使用序列
>  *   下面以飞盘工厂为示例：
``` 
//最基本的实例化，ps:在camera下面给diskPrefabs添加resource下面的Disk
public GameObject diskPrefab;
GameObject disk = GameObject.Instantiate<GameObject>(diskPrefab);

//存放正在使用的飞盘
private List<GameObject> usingUFOs;

//存放回收的不使用的飞盘
private List<GameObject> uselessUFOs;

//工厂的单例化，保证只有一个工厂
public static UFOFactory getFactory()
		{
			if (_UFOFactory == null)
			{
				_UFOFactory = new UFOFactory();//相当于创造一个空的工厂，里面有usingUFOs这些指针，但还没有分配空间
				_UFOFactory.uselessUFOs = new List<GameObject>();
				_UFOFactory.usingUFOs = new List<GameObject>();
			}
			return _UFOFactory;
		}

   
//从usingDisks回收已使用的飞盘
public void deleteUFO(GameObject UFO)
		{
			//Debug.Log ("deleteUFO");
			int index = usingUFOs.FindIndex(x => x == UFO);//找到usingUFOs里面要回收的UFOs放回uselessUFOs
			uselessUFOs.Add(UFO);
			usingUFOs.RemoveAt(index);
		}

//创建飞盘
public List<GameObject> creatUFOs(int UFOCount)
		{
			for (int i = 0; i < UFOCount; i++)
			{	
				
				if (uselessUFOs.Count == 0)
				{
					GameObject UFO = GameObject.Instantiate<GameObject>(UFOprefab);

					//UFO.SetActive (false);
					usingUFOs.Add(UFO);

				}
				else//从uselessUFO拿出来，不额外创建
				{
					GameObject UFO = uselessUFOs[0];

					uselessUFOs.RemoveAt(0);
					usingUFOs.Add(UFO);
					//Debug.Log ("create UFO from uselessUFO");
				}
			}
			return this.usingUFOs;
		}


```
#### 反思与不足之处
>  * 过度单例化
>   *  一开始打算让所有类都实现单例化，例如Controller大类有一个getController()的单例化函数，Scene大类有一个getScene()的单例化函数
>   *  但出现问题是如果所有csharp文件的类都用了单例化，意味着没有代码可以负载在object上运行，因此要改进的是创建一个不用单例化的main() 
>   *  main大类继承Monobehavior。
>  * 函数命名
>   *  一个csharp文件只有下面有与其名字相同的public class xxx:monobehavior{}类时才能被加载到object上

#### 下面为完整代码，还有不足之处有待后期改进  
##### Scene.cs
```
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using com.UFO;


namespace com.UFO{
	public class Scene{
		public static Scene _Scene;
		public List<GameObject> _usingUFO;

		private bool showChangeScoreEnable;
		private bool showChangeStatusEnable;
		private int _score;
		private int _status;
		private Vector3 throwPosition;
		private double throwSpeed;
		private Vector3 throwDirection;


		private int UFOspeed;

		public bool getshowChangeStatusEnable(){
			return showChangeStatusEnable;
		}

		public static Scene getScene(){
			if (_Scene == null){
				_Scene = new Scene();
				_Scene._usingUFO= new List<GameObject>();
			}
			return _Scene;
		}
		public void reset(int round){//根据回合数重置飞边扔出的各种参数
			throwSpeed=1;
			//throwDirection = new Vector3(3f, 8f, 5f);
			throwDirection = new Vector3(3f, 0, 0);
			//throwPosition = new Vector3 (-5f, 3f, -15f);
			throwPosition = new Vector3 (-5f, 10f, 0);
		}

		public void throwUFO(List<GameObject> usingUFO){//扔飞碟
			_usingUFO=usingUFO;
			getScene().reset(Controller.getController().round);
			for(int i=0;i<_usingUFO.Count;i++){//向图中扔飞碟,设置方向，大小，颜色等属性

				usingUFO [i].transform.position =new Vector3(throwPosition.x,throwPosition.y,throwPosition.z);
				//usingUFO [i].transform.position = throwPosition;
				Rigidbody rigibody;
				rigibody = usingUFO[i].GetComponent<Rigidbody>();
				rigibody.WakeUp();
				rigibody.useGravity = true;
				//rigibody.AddForce(throwDirection* Random.Range((float)throwSpeed * 5, (float)throwSpeed* 8) / 5, ForceMode.Impulse);
				rigibody.AddForce(throwDirection, ForceMode.Impulse);
				rigibody.velocity = throwDirection;
				//usingUFO[i].SetActive(true);
				Debug.Log ("throwUFO");

			}
		}
		public void  deleteUFO(GameObject UFO){
			UFO.GetComponent<Rigidbody> ().Sleep ();
			UFO.GetComponent<Rigidbody> ().useGravity = false;
			UFO.transform.position = new Vector3 (0f, 20f, 0f);

		}

		public void showChangeScore(int score){
			_score = score;
			showChangeScoreEnable = true;
		}
		public void showChangeStatus(int Status){
			_status = Status;
			showChangeStatusEnable = true;
		}
	}
}


public class SceneMono:MonoBehaviour{
	private GUIStyle fontStyle=new GUIStyle();
	private float time;
	Scene _scene;

	// Use this for initialization
	void Start () {
		fontStyle.fontSize = 40;
		_scene = Scene.getScene ();
	}

	// Update is called once per frame
	void OnGUI(){

		GUI.Label (new Rect (Screen.width / 2, Screen.height / 2-100, 100, 50), model.getModel().Score.ToString(),fontStyle);


		if ( Scene.getScene().getshowChangeStatusEnable()) {
			if (model.getModel().Status == -1) {
				GUI.Label (new Rect (Screen.width / 2, Screen.height / 2, 100, 50), "fail", fontStyle);
			} else if (model.getModel().Status == 1) {
				GUI.Label (new Rect (Screen.width / 2, Screen.height / 2, 100, 50), "win", fontStyle);
			}
		}

	}

	void Update () {
		time += Time.deltaTime;
		if (time >1.5) {//每1.5秒发射一个
			Debug.Log ("time: " + time.ToString());
			time=time-1.5f;
			Controller.getController ().throwUFO (1);
		}

		Ray ray = Camera.main.ScreenPointToRay (Input.mousePosition);
		RaycastHit hit;
		if (Physics.Raycast (ray, out hit) && hit.collider.gameObject.tag == "UFO") {
			Controller.getController ().deleteUFO (hit.collider.gameObject);
			model.getModel ().addScore (10);
			Controller.getController().addScore(10);
		}
	}
}

```
##### Model.cs
```
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using com.UFO;

public class model{//进行成绩等游戏逻辑处理

	public int Score{ get;set;}
	public int Status{ get;set;}//-1为输，1为赢，0表示继续

	public static model _model;
	public static model getModel(){
		/*if (_model == null){
			//_model = new model();

		}*/
		return _model;
	}
	public void addScore(int addNum){
		//model.getModel().Score = addNum+model.getModel().Score;
		this.Score+=addNum;
		Debug.Log ("score: " + this.Score);

		Scene.getScene ().showChangeScore (Score);
		if (model.getModel().Score >= 100) {//胜利
			Status = 1;
			Scene.getScene ().showChangeStatus (1);
		}
	}
	public void subScore(int subNum){
		Score -= subNum;
		Scene.getScene ().showChangeScore (Score);

		if (Score < 0) {//失败
			Status = -1;
			Scene.getScene ().showChangeStatus (-1);
		}

	}

	// Use this for initialization
	void Start () {
		//_model= gameObject.AddComponent<model>() as model;
		Status = 0;
	}
	
	// Update is called once per frame
	void Update () {
		
	}
}

```
##### Controller.cs
```
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using com.UFO;

namespace com.UFO{
	public class Controller  {



		public int round{ get; set;}//从1开始

		public static Controller _Controller;
		private List<GameObject> usingUFOs;
		private List<GameObject> uselessUFOs;

		public List<GameObject> getUsingUFOs(){
			return usingUFOs;
		}
		public List<GameObject> getUselessUFOs(){
			return uselessUFOs;
		}

		public static Controller getController(){
			if (_Controller == null){
				_Controller = new Controller();
				_Controller.usingUFOs=new List<GameObject>();
				_Controller.uselessUFOs = new List<GameObject> ();
			}
			return _Controller;
		}

		public void addScore(int addNum){
			model.getModel ().addScore (addNum);
		}
		public void subScore(int subNum){
			model.getModel ().subScore (subNum);
		}
		public void throwUFO(int num){
			//先从工厂里面拿飞盘

			usingUFOs=UFOFactory.getFactory().creatUFOs(num);
			Scene.getScene ().throwUFO (usingUFOs);

		}
		public void deleteUFO(GameObject UFO){
			UFOFactory.getFactory().deleteUFO(UFO);
			Scene.getScene().deleteUFO(UFO);
		}



		// Use this for initialization


}
public class ControllerMono:MonoBehaviour{
		public GameObject UFOPrefab;
		public UFOFactory _UFOFactory;
		private List<GameObject> _usingUFOs;
		private List<GameObject> _uselessUFOs;
		void Start () {
			_UFOFactory = UFOFactory.getFactory();
			_UFOFactory.UFOprefab = this.UFOPrefab;

		}

		// Update is called once per frame
		void Update () {
			//Debug.Log ("ControllerUpdate");
			_usingUFOs=_UFOFactory.getUsingUFOs();
			_uselessUFOs = _UFOFactory.getuselessUFOs ();
			_usingUFOs= UFOFactory.getFactory ().getUsingUFOs ();
			_uselessUFOs= UFOFactory.getFactory ().getuselessUFOs ();
			//Debug.Log("usingUFOs.Count: "+usingUFOs.Count);
			//Debug.Log("uselessUFOs.Count: "+uselessUFOs.Count);
			for (int i = 0; i < _UFOFactory.getUsingUFOs().Count; i++) {
				if (_UFOFactory.getUsingUFOs()[i].transform.position.y < -10) {//判断出界,回收
					//model.getModel().subScore(10);
					Controller.getController().deleteUFO(_UFOFactory.getUsingUFOs()[i]);

				}
			}

		}
	}

} 


```
##### UFOFactory.cs
```
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using com.UFO;

namespace com.UFO{

	public class UFOFactory : System.Object {

		public GameObject UFOprefab;
		private static UFOFactory _UFOFactory;
		private List<GameObject> usingUFOs;
		private List<GameObject> uselessUFOs;

		public List<GameObject> getUsingUFOs(){
			return usingUFOs;
		}
		public List<GameObject>  getuselessUFOs(){
			return uselessUFOs;
		}

		public static UFOFactory getFactory()
		{
			if (_UFOFactory == null)
			{
				_UFOFactory = new UFOFactory();//相当于创造一个空的工厂，里面有usingUFOs这些指针，但还没有分配空间
				_UFOFactory.uselessUFOs = new List<GameObject>();
				_UFOFactory.usingUFOs = new List<GameObject>();
			}
			return _UFOFactory;
		}

		public List<GameObject> creatUFOs(int UFOCount)
		{
			for (int i = 0; i < UFOCount; i++)
			{	
				
				if (uselessUFOs.Count == 0)
				{
					GameObject UFO = GameObject.Instantiate<GameObject>(UFOprefab);

					//UFO.SetActive (false);
					usingUFOs.Add(UFO);

				}
				else//从uselessUFO拿出来，不额外创建
				{
					GameObject UFO = uselessUFOs[0];

					uselessUFOs.RemoveAt(0);
					usingUFOs.Add(UFO);
					Debug.Log ("create UFO from uselessUFO");
				}
			}
			return this.usingUFOs;
		}
		public void deleteUFO(GameObject UFO)
		{
			Debug.Log ("deleteUFO");
			int index = usingUFOs.FindIndex(x => x == UFO);//找到usingUFOs里面要回收的UFOs放回uselessUFOs
			uselessUFOs.Add(UFO);
			usingUFOs.RemoveAt(index);
		}
	}

}
public class UFOFactoryMono : MonoBehaviour
{
	UFOFactory _UFOFactory;
	public GameObject UFOprefab;

	void Awake()
	{
		_UFOFactory = UFOFactory.getFactory ();
		_UFOFactory.UFOprefab = UFOprefab;
	}
	void Update(){
		Debug.Log ("FactoryUpdate");
	}
}

```



