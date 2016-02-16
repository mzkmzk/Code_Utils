#HapLab

##1. 项目搭建

###1.1 安装环境

1. apache
2. PHP >= 5.5.9 
3. MySQL
4. 建议IDE: PHPStorm

###1.2 安装HapLab

1. clone haplab : git@turing.haplox.net:platform/haplab.git
2. 更改`php.ini`

		disaply_eeror = on //建议
		errors_log = 指定路径 //建议
		date.timezone = Asia/Shanghai //必须
3. apche加载`mod_rewrite`模块(一般都没开),`OpenSSL模块 PDO模块 Mbstring模块 Tokenizer模块`(默认开)	
		
	加载方式
	
	window/Mac : `sudo vim /etc/httpd.conf`	 -> 去掉注释`LoadModule rewrite_module libexec/apache2/mod_rewrite.so`
	
	Ubuntu: 
		1. sudo a2enmod rewrite//加载重定向模块
		2. /etc/apche2/apache.conf里的所有AllowOverride None修改为AllowOverride All
3. 权限设置

	1. sudo chmod -R 777 项目/storage;
	2. sudo chmod -R 777 项目/bootstrap;
4. 项目根目录创建.env	

	例子:
	
		APP_ENV=local
		APP_DEBUG=true
		APP_KEY=7JeT4sv8H12QyKCW3vh5PjqthzOVZEAS


		DB_HOST=127.0.0.1
		DB_DATABASE=数据库名
		DB_USERNAME=数据账号
		DB_PASSWORD=数据库密码

		CACHE_DRIVER=file
		SESSION_DRIVER=file
		QUEUE_DRIVER=sync


		MAIL_DRIVER=smtp
		MAIL_HOST=smtp.exmail.qq.com
		MAIL_NAME=HapLab //发送人姓名
		MAIL_PORT=465
		MAIL_USERNAME=邮箱账号 //eg:maizk@haplox.com
		MAIL_PASSWORD=邮箱密码
		MAIL_ENCRYPTION=ssl
5. 访问 `localhost/项目名/public`	

##2. 目录说明

	├── app
	│   ├── Http					###routes.php为路由定义
	│   │   ├── Controllers  			###Controller层
	│   │   ├── Middleware
	│   │   └── Requests
	|	│   └── Utils					###自定义全局函数
	|   ├── Models                 ###模型层
	├── config                     ###系统配置文件
	├── database					###数据库
	│   ├── factories					###工厂定义
	│   ├── migrations				###数据库表定义
	│   └── seeds						###数据填充
	├── public						###JS/CSS	
	│   ├── css
	│   │   └── messenger-1.4.2
	│   └── js
	│       ├── amcharts
	│       ├── entity				###各实体JS
	│       └── messenger-1.4.2
	├── resources
	│   └── views                  ###视图定义
	├── storage          			###仓库存储图片/日志   
	│   ├── Medical_Record_Pictures### 病历存放
	│   ├── Reports                ### 报告存放
	│   ├── app
	│   ├── framework
	│   │   ├── cache
	│   │   ├── sessions
	│   │   └── views
	│   └── logs
	└── tests						###测试用例

##3. 数据库及设计

###3.1 数据库

[领域模型图,Password:Find K](https://www.processon.com/view/link/5636d1b7e4b06c4efe5e6176)

###3.2 流程图 

[系统流程图,Password:Find K](https://www.processon.com/view/link/56527eeae4b0bdf839630c7e)

passwd : HapLab2015!

##4 功能点介绍

###4.1 基本CRUD

1. 路由设置(`项目路径/app/Http/routes.php`):

		Route::group(['prefix' => 'Controller'],function(){

    		Route::group(['prefix' => 'User_Controller'],function(){
        		Route::match(['get','post'],'login','User_Controller@login');
        		Route::match(['get','post'],'insert','User_Controller@insert');
        		Route::match(['get','post'],'update','User_Controller@update');
        		Route::match(['get','post'],'query','User_Controller@query');
        		Route::match(['get','post'],'delete','User_Controller@delete');
        		Route::match(['get','post'],'restore','User_Controller@restore');
    		});
    		...
    	});
    所有的CRUD,基本的URL都为`../Controller/实体名称_Controller/方法`
    
2. Controller设置(`项目路径/app/Http/Controllers`)
	
	`Base_Controller`: 所有需要CRUD的Controller,需要继承Base_Controller
	
	    class Base_Controller extends Controller{

            //继承的Controller,必须在构造方法中,注入相对应的实体Model
            protected  $model = null;

            public function base_query(Request $request){

            //搜索名字,返回的$request 会带有`IDs`,表明符合病人名字匹配的当前实体ID.
            //能根据病人名字搜索到的实体Model,都会有`get_IDs_by_patientName`方法,进行逐级递归查找
            if(isset($request['patient_name'])){
                $request = $this->model->get_IDs_by_patientName($request);
            }
    
            $model =  $this->model->query();
    
            if(isset($request['IDs'])){
                $model = $model->whereIn('id',$request['IDs']);
            }
    
            //获取实体属性,对where查询进行过滤非本实体的属性.
            $attribute_one = get_model_attribute($this->model);
            foreach($request->all() as $key => $value){
                if (array_key_exists($key,$attribute_one) ==true){
                    $model = $model->where($key,$value);
                }
            }
    
             $page_num = null;
            if($request['page_size'] !=null){
                $page_num = $request['page_size'];
            }else {
                $page_num = 20;
            }
    
            $model = $model->orderBy("updated_at","desc")
                ->Paginate($page_num)
                ->toJson();
    
           return $model;
        }
    
        protected function base_insert(Request $request){
    
            $result_data = array();
    
            $attribute_one = get_model_attribute($this->model);
    
            $request_all = $request->all();
    
            $array_length = object_all_array($request_all);
    
            //单个 else批量处理
            if($array_length == 0){
                foreach($request->all() as $key => $value){
                    if (array_key_exists($key,$attribute_one) ==true){
                        $this->model[$key] = $value;
                    }
                }
                $result["result"] = $this->model->save();
            }else {
                for($index =0;$index<$array_length;$index++){
                    $attribute = array();
                    foreach($request_all as $key => $value){
                        if (array_key_exists($key,$attribute_one) ==true){
                            $this->model[$key] = $value[$index];
                            $attribute[$key] = $value[$index];
                        }
                    }
    
                    $create_model = $this->model->create($attribute);
                    array_push($result_data,$create_model);
    
                    $result["result"] = !is_null($create_model);
                }
            }
            $result['data'] = $result_data;
            return $result;
        }
    
        public function base_update(Request $request){
            //根据id,找到相应实体
            $this->model = $this->model->find($request->get('id'));
            //获取实体的属性,用来过滤非实体属性的更新
            $attribute_one = get_model_attribute($this->model);
            foreach($request->all() as $key => $value){
                if (array_key_exists($key,$attribute_one) ==true){
                    $this->model[$key] = $value;
                }
            }
            $result["result"] = $this->model->save();
            return $result;
        }
    
        public function base_delete(Request $request){
            $result["result"] = $this->model
                ->where("id",$request->get("id"))
                ->delete();
            return $result;
        }
    
        public function base_restore(Request $request){
            $result["result"] = $this->model->withTrashed()
                ->where("id",$request->get("id"))
                ->restore();
            return $result;
        }
    
        //默认执行的query
        protected function query(Request $request){
            return $this->base_query($request);
        }
    
        //默认执行的insert
        protected function insert(Request $request){
            return json_encode($this->base_insert($request));
        }
    
        //默认执行的update
        public function update(Request $request){
            return json_encode($this->base_update($request));
        }
    
        //默认执行的delete
        public function delete(Request $request){
            return json_encode($this->base_delete($request));
        }
    
        //默认执行的restore
        public function restore(Request $request){
            return json_encode($this->base_restore($request));
        }
    
	eg: Sample_Controller:
	
        class Sample_Controller extends Base_Controller{

        /**
         * 给Base_Controller注入Model
         */
        public function __construct(){
            $this->model = new Sample_Model();
        }
    
        /**
         * 覆盖父类insert方法,重新实现.
         * @param Request $request
         * @return string
         */
        public function insert(Request $request){
            $result = $this->base_insert($request);
            //因为预处理中有用的属性太少,在增加样本的时候,自动生成预处理记录
            for($index = count($result['data']);$index--;){
                Blood_Dispose_Record_Model::create(['process_user_ID'=>'0','sample_ID'=>$result['data'][$index]->id]);
            }
            return json_encode($result);
        }
    
        /**
         * 主要用于,批量增加数据时,在确定上级关系时,只需填写名字就会自动搜索匹配名字的上级.
         * @param Request $request
         * @return array
         */
        public function query_id_by_patientName(Request $request){
    
            //存放返回的label=>value数组.
            $result =array();
    
            //讲输入框输入的字符串转换到patient_name中
            $request['patient_name'] = $request['term'];
    
            //查询匹配名字的上级
            $patient_controller = new Patient_Controller();
            $result_model_collection_json = $patient_controller->query($request);
    
            //将查询的结果放入$result中
            $result_model_collection = json_decode($result_model_collection_json);
            $model_collection = $result_model_collection->data;
            if($result_model_collection->total !=0)
                foreach( $model_collection as $model_one){
                    //label属性为提示的文字,value代表上级的id.
                    $obj = new \label_value();
                    $obj->label = $model_one->name ." 创建时间: " .  $model_one->created_at ;
                    $obj->value = trim($model_one->id);
                    array_push($result,$obj);
                }
            return $result;
        }
        
3. Model设置        

    举例Sample_Model
    ```php
    class Sample_Model extends Base_Model{

        //数据库表名
        protected $table = 'Samples';
    
        //Model转换为json时,会增加属性patient_name => '会执行getPatientNameAttribute方法返回的结果'
        public $appends = ['patient_name'];
    
        //指定下级的关系,现在主要用于数据导出时,根据model名找到下机级联的实体表,然而进行数据读取
        // 数据读取的Controller App\Http\Controllers\Excel/All_Excel;
        public static $next_relationships = ['blood_dispose_records'];
    
        //声明上级关系
        public function patient(){
            return $this->belongsTo('App\Models\Patient_Model','patient_ID','id');
        }
    
        //声明下级关系
        public function blood_dispose_records(){
            return $this->hasMany('App\Models\Blood_Dispose_Record_Model','sample_id','id');
        }
    
        //声明下级关系
        public function report(){
            return $this->hasMany('App\Models\Report_Model','sample_id','id');
        }
    
        //用于获取Model的时候,把病人名字也获取了
        public function getPatientNameAttribute(){
            if($this->patient !=null){
                return $this->patient->name ;
            }else {
                return "";
            }
        }
    
        /**
         * 根据病人的姓名获取本实体中能匹配的ID
         * 具体思路是调用上级Model的get_IDs_by_patientName方法,直接递归到Patient实体中,就能获取上级匹配的IDs,然后把上级匹配的IDs和本实体筛选
         * @param Request $request
         * @return Request
         */
        public function get_IDs_by_patientName(Request $request){
            $patient_model = new Patient_Model();
            $request = $patient_model->get_IDs_by_patientName($request);
            $sample_model = new Sample_Model();
            $sample_model = $sample_model
                ->whereIn("patient_id",$request['IDs'])
                ->get(['id']);
    
            $IDs = array();
            foreach($sample_model as $sample_model_one){
                array_push($IDs,$sample_model_one->id);
            }
    
            $request['IDs'] = $IDs;
            return $request;
        }
    
        /**
         * 判断有无下一操作.
         * 主要用在邮件提醒中.主要思路为在每个Model中设置判断有无下级数据
         * @param $blood_dispose_record_model
         * @param $check_type
         * @return bool|void
         */
        public static function query_has_next_operate($model,$check_type){
            $next_model_array = $model->blood_dispose_records;
            if(count($next_model_array) == 0){
                return "预处理没有记录";
            }
            return Blood_Dispose_Record_Model::query_has_operate($next_model_array,$check_type);
    
        }
    
    }
    ```
    
4. 我