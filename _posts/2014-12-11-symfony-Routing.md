---
layout: post
category: symfony
tags: [php,symfony,routing]
---
#Routing

##学习目的

* Create complex routes that map to controllers

* Generate URLs inside templates and controllers

* Load routing resources from bundles (or anywhere else)

* Debug your routes

####一个annotation的示例
  
    // src/AppBundle/Controller/BlogController.php
    namespace AppBundle\Controller;

    use Symfony\Bundle\FrameworkBundle\Controller\Controller;
    use Sensio\Bundle\FrameworkExtraBundle\Configuration\Route;

    class BlogController extends Controller
    {
            /**
             * @Route("/blog/{slug}")
             */
            public function showAction($slug)
            {
                                          // ...
            }
    }

该路由指向/blog  传入一个$slug的参数

##创造Routes
在config文件夹中的config.yml文件中包含入routing.yml文件
    
    app/config/config.yml
    framework:
         ...
        router: { resource: "%kernel.root_dir%/config/routing.yml" }

##basic route configuration

    class MainController extends Controller
    {
            /**
             * @Route("/")
             */
             public function homepageAction()
             {
                                          // ...
             }
    }
    
一个基础的路由指向  (/)  maps it to the AppBundle:Main:homepage controller

###路由参数

可以在给路由分配参数的时候指定默认值

    /**
     * @Route("/blog/{page}", defaults={"page" = 1})
      */
      public function indexAction($page)
      {
              // ...
      }
      

    /**
     * @Route("/blog/{slug}")
     */
    public function showAction($slug)
    {
        // ...
    }

当出现2个同样的路由请求时，但是参数类型不同例如{page} = 2,{slug} = "my-blog-post" ，可以对参数进行类型限制

    /**
     * @Route("/blog/{page}", defaults={"page": 1}, requirements={"page": "\d+"})
      */
      public function indexAction($page)
      {
              // ...
      }

通过\d+限制了page只可以接受到为数字的值

也可以对参数进行选择限制，规定路由后所带的参数的值,如下例中的 限制参数的值只可以为en和fr

    class MainController extends Controller
    {
        /**
         * @Route("/{_locale}", defaults={"_locale": "en"}, requirements={"_locale": "en|fr"})
         */
        public function homepageAction($_locale)
                              {
                                      }
    }

该例子中$_locale的值默认为en ，限制为只可以为en和fr的值

同样可以限制HTTP请求的方式
    
    // src/AppBundle/Controller/MainController.php
    namespace AppBundle\Controller;

    use Sensio\Bundle\FrameworkExtraBundle\Configuration\Method;
    // ...

    class MainController extends Controller
    {
    /**
     * @Route("/contact")
     * @Method("GET")
     */
    public function contactAction()
    {
        // ... display contact form
    }

    /**
     * @Route("/contact")
     * @Method("POST")
     */
    public function processContactAction()
    {
        // ... process contact form
    }
    }

可以限制一个请求为get 方式 或者为 post 方式 ，可以充分利用http协议中的请求动作,get展示表单，post方式接收处理表单

在用@Method()进行请求动作判断时需要 use Sensio\Bundle\FrameworkExtraBundle\Configuration\Method;


###可以Match a Route Based on the Host

###在配置文件中限定一个路由的访问条件，用condition项 
    
    contact:
    path:     /contact
    defaults: { _controller: AcmeDemoBundle:Main:contact }
    condition: "context.getMethod() in ['GET', 'HEAD'] and request.headers.get('User-Agent') matches '/firefox/i'"

这个condition在缓存中生成的php代码如下
    
    if (rtrim($pathinfo, '/contact') === '' && (
            in_array($context->getMethod(), array(0 => "GET", 1 => "HEAD"))
            && preg_match("/firefox/i", $request->headers->get("User-Agent"))
            )) {
    // ...
    }

###一个完整的routing例子

    class ArticleController extends Controller
    {
        /**
         * @Route(
         *     "/articles/{_locale}/{year}/{title}.{_format}",
         *     defaults= {"_format": "html"},
         *     requirements= {"_locale": "en|fr", "_format": "html|rss", "year": "\d+"}
         * )
         */
        public function showAction($_locale, $year, $title)
        {
        }
    }

*注意defaults 后面跟的是= 而不是 :  ,因为官网貌似写错了

###特殊的路由参数
    
    _controller:As you've seen, this parameter is used to determine which controller is executed when the route is matched;

    _format:Used to set the request format;

    _locale:Used to set the locale on the request

可以在config文件夹中routing.yml文件中定义_controller

    app/config/routing.yml
        hello:
            path:     /hello
            defaults: { _controller: acme.hello.controller:indexAction }

###路由前缀

可以在定义路由的时候加上前缀
    
    app/config/routing.yml
    app:
        resource: "@AppBundle/Controller/"
        type:     annotation
        prefix:   /site

###可以为路由指定域名

    acme_hello:
        resource: "@AcmeHelloBundle/Resources/config/routing.yml"
        host:     "hello.example.com"
        
##对路由进行debug的测试

    $ php app/console debug:router

根据路由名称，获取相关信息

    $ php app/console debug:router 路由名称

可以写一个url进行测试看匹配到什么路由

    $ php app/console router:match /blog/my-latest-post

获取一个uri对应的路由

    $param = $this -> get('router') ->match('/demo/hello/2014');
    var_dump($param);
    
结果为

    array (size=3)
      '_controller' => string 'Acme\DemoBundle\Controller\DemoController::helloAction' (length=54)
        'name' => string '2014' (length=4)
          '_route' => string '_demo_hello' (length=11)

获取一个路由对应的uri

    $uri = $this -> get('router') -> generate('_main_anni');
    var_dump($uri);

结果为
    
    string '/demo/abc' (length=9)

获取一个路由对应的url（在继承了baseController后就可以用generateURl方法）
    
    $url = $this  ->generateUrl('_demo_contact');
     echo "<br>";
    var_dump($url);

结果为

    string '/demo/contact' (length=13)

获取一个包含域名的地址如下

    $url = $this  ->generateUrl('_demo_contact',array(),true);
    echo "<br>";
    var_dump($url);

结果为

    string 'http://test.sy.com/demo/contact' (length=31)

如果没有继承symfony的baseController,可以获取container后进行获取url

    $url = $this->container->get('router')->generate(
        'blog_show',
        array('slug' => 'my-blog-post')
        );

当在js中使用ajax请求的时候获取uri地址的时候可以使用FOSJsRoutingBundle

当加载该bundle后可以直接在js中使用router

    var url = Routing.generate(
        'blog_show',
            {"slug": 'my-blog-post'}
            );

在模板中获取一个uri

    <a href="{{ url('blog_show', {'slug': 'my-blog-post'}) }}">
      Read this blog post.
    </a>

















