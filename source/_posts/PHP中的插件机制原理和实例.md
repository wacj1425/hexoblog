---
title: PHP中的插件机制原理和实例
date: 2018-02-03 16:19:05
tags: [php]
categories: [php开发,学习笔记]
---
> 这篇文章主要介绍了PHP中的插件机制原理和实例,文中例子主要借鉴了网上一些网友的方式做了稍微的改造,需要的朋友可以参考下

这篇文章主要介绍了PHP中的插件机制原理和实例,文中例子主要借鉴了网上一些网友的方式做了稍微的改造,需要的朋友可以参考下

**从一个插件安装到运行过程的角度来说，主要是三个步骤：**

1. 插件安装（把插件信息收集进行采集和记忆的过程，比如放到数据库中或者XML中）
2. 插件激活（打开插件，让监听插件的地方开始进行调用）
3. 插件运行（插件功能的实现）

**从一个插件的运行上来说主要以下几点：**

1. 插件的动态监听和加载（插件的信息获取）
2. 插件的动态触发（插件的运行）

**一个完善的插件系统主要包括以下：**

1. 插件安装及卸载
2. 插件打开与关闭
3. 插件信息获取
4. 插件调度（插件经理）
5. 插件主体

**在程序的编写上主要实现以下：**

1. 插件的注册和初始化
2. 判断激活条件
3. 钩子激活
4. 运行插件

**实例代码：**

```
<?php
/** 
* PluginManager Class 
* 
* 插件机制的实现核心类 
* 
* @link http://www.jb51.net/ 
*/ 
class PluginManager 
{ 
  /** 
   * 监听已注册的插件 
   * 
   * @access private 
   * @var array 
   */ 
  private $_listeners = array(); 
   /** 
   * 构造函数 
   * 
   * @access public 
   * @return void 
   */ 
  public function __construct() 
  { 
    #这里$plugin数组包含我们获取已经由用户激活的插件信息 
   #为演示方便，我们假定$plugin中至少包含 
   #$plugin = array( 
    #  'name' => '插件名称', 
    #  'directory'=>'插件安装目录' 
    #); 
   
 
   // $plugins = get_active_plugins();#这个函数请自行实现 
 
    //函数实现后的最终数据结构效果如下
    $plugins=array(array("directory"=>"demo",
    "name"=>"DEMO"));
 
 
    if($plugins) 
    { 
      foreach($plugins as $plugin) 
 
      {//假定每个插件文件夹中包含一个actions.php文件，它是插件的具体实现 
        if (@file_exists(STPATH .'plugins/'.$plugin['directory'].'/actions.php')) 
        { 
          include_once(STPATH .'plugins/'.$plugin['directory'].'/actions.php'); 
          $class = $plugin['name'].'_actions'; 
          if (class_exists($class)) 
          { 
            //初始化所有插件 
            //$this 是本类的引用
            new $class($this); 
          } 
        } 
      } 
    } 
    #此处做些日志记录方面的东西 
  } 
 
  /** 
   * 注册需要监听的插件方法（钩子） 
   * 
   * @param string $hook 
   * @param object $reference 
   * @param string $method 
   */ 
  function register($hook, &$reference, $method) 
  { 
    //获取插件要实现的方法 
    $key = get_class($reference).'->'.$method; 
    //将插件的引用连同方法push进监听数组中 
    $this->_listeners[$hook][$key] = array(&$reference, $method); 
    #此处做些日志记录方面的东西 
  } 
  /** 
   * 触发一个钩子 
   * 
   * @param string $hook 钩子的名称 
   * @param mixed $data 钩子的入参 
   *  @return mixed 
   */ 
  function trigger($hook, $data='') 
  { 
    $result = ''; 
    //查看要实现的钩子，是否在监听数组之中 
    if (isset($this->_listeners[$hook]) && is_array($this->_listeners[$hook]) && count($this->_listeners[$hook]) > 0) 
    { 
      // 循环调用开始 
      foreach ($this->_listeners[$hook] as $listener) 
      { 
        // 取出插件对象的引用和方法 
        $class =& $listener[0]; 
        $method = $listener[1]; 
        if(method_exists($class,$method)) 
        { 
          // 动态调用插件的方法 
          $result .= $class->$method($data); 
        } 
      } 
    } 
    #此处做些日志记录方面的东西 
    return $result; 
  } 
} 
 
define(STPATH, "./");
 
$pluginManager=new PluginManager();
 
$pluginManager->trigger("demo");
```

**demo插件文件:**

```
<?php
/**
 * 这是一个Hello World简单插件的实现
 *
 * @link    http://www.jb51.net/
 */
/**
 *需要注意的几个默认规则：
 *  1. 本插件类的文件名必须是action
 *  2. 插件类的名称必须是{插件名_actions}
 */
class DEMO_actions
{
  //解析函数的参数是pluginManager的引用
  function __construct(&$pluginManager)
  {
    //注册这个插件
    //第一个参数是钩子的名称
    //第二个参数是pluginManager的引用
    //第三个是插件所执行的方法
    $pluginManager->register('demo', $this, 'say_hello');
  }
 
  function say_hello()
  {
    echo 'Hello World';
  }
}
```