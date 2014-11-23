---
layout: post
category: symfony
tags: [php,symfony,controller]
---
#Controller

symfony中controller就和HTTP一样，有了request后，必须要有respose.

response可以是一个Response object, an HTML page, an XML document, a serialized JSON array, an image, a redirect, a 404 error or anything else
    
    示例
    use Symfony\Component\HttpFoundation\Response;

    public function helloAction()
    {
            return new Response('Hello world!');
    }
##一个请求的基本流程

    1. Each request is handled by a single front controller file (e.g. app.php or app_dev.php) that bootstraps the application; 每一个请求都由前台控制器进行任务分发
    2. The Router reads information from the request (e.g. the URI), finds a route that matches that information, and reads the _controller parameter from the route; 经过路由进行请求分析读取controller信息
    3. The controller from the matched route is executed and the code inside the controller creates and returns a Response object;和路由匹配的controller执行业务逻辑代码，创建并返回一个Response
    4. The HTTP headers and content of the Response object are sent back to the client.根据HTTP头信息返回到客户端


####出现的问题

1. 返回一个Response对象后，访问页面进行报错，提示不是一个Response对象

####解决方法

2. 加载的命名空间出现问题，应该加载use Symfony\Component\HttpFoundation\Response; 而不是加载use Symfony\Component\BrowserKit\Response; 


##映射一个URL到Controller
方式1:通过annotation方式在注释中规定routing
    在config文件中指定已annotation方式进行路由发放，可以指定路由前缀,可以传入$name的变量,可以传入多个变量。在方法中定义的参数必须要有值传入，路由中定义的变量，可以不传到方法的参数中
    _demo_secured:
    resource: "@AcmeDemoBundle/Controller/SecuredController.php"
    type:     annotation
    prefix:   /demo

    /**
     * @Route("/hello/{name}", name="hello")
     */
    public function indexAction($name)
    {
        return new Response('<html><body>Hello '.$name.'!</body></html>');
    }

方式2:通过在config目录下的routing.yml文件中进行配置，直接指定到控制器下的方法

    _anni_xman:
    path: /xman
    defaults: {_controller: AcmeDemoBundle:Anni:xman }

##在控制器中获取request的信息，可以直接在参数中传入Request类的对象参数，获取get传入的page值
    
    use Symfony\Component\HttpFoundation\Request;

    public function indexAction($firstName, $lastName, Request $request)
    {
            $page = $request->query->get('page', 1);

                // ...
    }

##基础控制器类

继承了基础控制类后，可以通过$this -> get()来获取symfony的container中的其他service，查看container中包含的所有的sevice可以使用命令
    
    $ php app/console debug:container

继承基础控制器，获取其他的service的代码
    
    namespace AppBundle\Controller;

    use Symfony\Bundle\FrameworkBundle\Controller\Controller;

    class HelloController extends Controller
    {
            // ...

            $templating = $this->get('templating');

            $router = $this->get('router');

            $mailer = $this->get('mailer');
    }

##跳转

使用redirect()方法,通过generateUrl方法来获取URL，下例中的homepage指的是rout的name属性

    public function indexAction()
    {
            return $this->redirect($this->generateUrl('homepage'),200);
    }

可以使用RedirectResponse类进行跳转

    use Symfony\Component\HttpFoundation\RedirectResponse;

    return new RedirectResponse($this->generateUrl('homepage'));

##加载模板

使用$this -> render()进行加载
    
    // renders app/Resources/views/Hello/index.html.twig
    return $this->render('Hello/index.html.twig', array('name' => $name));

或者

    // renders app/Resources/views/Hello/Greetings/index.html.twig
    return $this->render('Hello/Greetings/index.html.twig', array('name' => $name));

##管理错误页面

可以通过丢出Exception的方式
    
    public function indexAction()
    {
        // retrieve the object from database
        $product = ...;
        if (!$product) {
              throw $this->createNotFoundException('The product does not exist')
        }

        return $this->render(...);
    }
    
##进行session操作

    use Symfony\Component\HttpFoundation\Request;

    public function indexAction(Request $request)
    {
        $session = $request->getSession();
    
        // store an attribute for reuse during a later user request
        $session->set('foo', 'bar');
    
        // get the attribute set by another controller in another request
         $foobar = $session->get('foobar');

        // use a default value if the attribute doesn't exist
      $filters = $session->get('filters', array());
    }


####flash message 可以使用session保存一个存在时间只是一个请求时间的session信息
    
    use Symfony\Component\HttpFoundation\Request;
    
    public function updateAction(Request $request)
    {
        $form = $this->createForm(...);

        $form->handleRequest($request);

        if ($form->isValid()) {
            // do some sort of processing

            $request->getSession()->getFlashBag()->add(
                    'notice',
                    'Your changes were saved!'
                    );

            return $this->redirect($this->generateUrl(...));
        }

        return $this->render(...);
    }

这是一个获取表单进行处理后进行跳转前，保存了一个flash message信息，在跳转的模板里面可以直接使用 

    {% for flashMessage in app.session.flashbag.get('notice') %}
    <div class="flash-notice">
    {{ flashMessage }}
    </div>
    {% endfor %}

一个请求之后flash message 就消失

## Response Object
    use Symfony\Component\HttpFoundation\Response;

    // create a simple Response with a 200 status code (the default)
    $response = new Response('Hello '.$name, Response::HTTP_OK);

    // create a JSON-response with a 200 status code
    $response = new Response(json_encode(array('name' => $name)));
    $response->headers->set('Content-Type', 'application/json');

有多种reponse的对象，JsonResponse，FileResponse , StreamedReponse


##Request Object
这是个获取请求的对象包含get，post等请求信息

    use Symfony\Component\HttpFoundation\Request;

    public function indexAction(Request $request)
    {
            $request->isXmlHttpRequest(); // is it an Ajax request?

            $request->getPreferredLanguage(array('en', 'fr'));

            $request->query->get('page'); // get a $_GET parameter

            $request->request->get('page'); // get a $_POST parameter
    }

##Create Static Page
创建一个静态页面（只需要一个route和一个template）
可以直接在config文件夹中的routing.yml进行配置
  
  acme_privacy:
        path: /privacy
        defaults:
            _controller: FrameworkBundle:Template:template
            template:    'AcmeBundle:Static:privacy.html.twig'

在模板中展示的方式

    {{ render(url('acme_privacy')) }}

可以将静态页面进行缓存

    acme_privacy:
        path: /privacy
        defaults:
                _controller:  FrameworkBundle:Template:template
                template:     'AcmeBundle:Static:privacy.html.twig'
                maxAge:       86400
                sharedAge:    86400

这个cache是HTTPCACHE

##跳转另一个控制器Forwarding to another Controller

通过$this->forward()进行跳转

    public function indexAction($name)
    {
            $response = $this->forward('AppBundle:Something:fancy', array(
                    'name'  => $name,
                            'color' => 'green',
                                ));

                // ... further modify the response or return it directly

                    return $response;
    }

在AppBundle中的something控制器的fancy就会被调用，同时传入$name,和$color 2个参数

    public function fancyAction($name, $color)
    {
            // ... create and return a Response object
    }



##By anni @Global City 2014-12-11






































































