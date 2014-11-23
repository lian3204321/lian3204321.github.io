---
layout: post
category: symfony
tags: [php,symfony,template]
---

#Template

使用的是twig模板

    * {{ ... }}: "Says something": prints a variable or the result of an expression to the template; 输出

    * {% ... %}: "Does something": a tag that controls the logic of the template; it is used to execute statements such as for-loops for example. 进行逻辑控制

    * {# ... #}: "Comment something": it's the equivalent of the PHP /* comment */ syntax. It's used to add single or multi-line comments. The content of the comments isn't included in the rendered pages. 进行注释

可以使用过滤器

    {{ title|upper }}

可以使用函数

    {% for i in 0..10 %}
        <div class="{{ cycle(['odd', 'even'], i) }}">
            <!-- some HTML here -->
        </div>
    {% endfor %}

##twig模板的缓存

在dev环境下打开debug模式时twig是不会做模板缓存的


##模板的继承和布局

布局一般使用block，因为一个网站的页头页脚不会经常发生变化.
下例为一个基础的布局
    
    {# app/Resources/views/base.html.twig #}
    <!DOCTYPE html>
    <html>
    <head>
    <meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
    <title>{% block title %}Test Application{% endblock %}</title>
    </head>
    <body>
    <div id="sidebar">
    {% block sidebar %}
    <ul>
    <li><a href="/">Home</a></li>
    <li><a href="/blog">Blog</a></li>
    </ul>
    {% endblock %}
    </div>

    <div id="content">
    {% block body %}{% endblock %}
    </div>
    </body>
    </html>

可以重写上面的基础模板
利用extends进行继承
    
    {# app/Resources/views/Blog/index.html.twig #}
    {% extends 'base.html.twig' %}

    {% block title %}My cool blog posts{% endblock %}

    {% block body %}
        {% for entry in blog_entries %}
                <h2>{{ entry.title }}</h2>
                <p>{{ entry.body }}</p>
        {% endfor %}
    {% endblock %}

这样就将base.html.twig中的title block  和 body block 进行了替换

###使用模板继承的注意点

    * If you use {% extends %} in a template, it must be the first tag in that template; 当使用extends继承模板时 一个tag标签必须是extends

    * The more {% block %} tags you have in your base templates, the better. Remember, child templates don't have to define all parent blocks, so create as many blocks in your base templates as you want and give each a sensible default. The more blocks your base templates have, the more flexible your layout will be; 一个基础的模板中，定义越多的block则你的基础模板就越灵活

    * If you find yourself duplicating content in a number of templates, it probably means you should move that content to a {% block %} in a parent template. In some cases, a better solution may be to move the content to a new template and include it (see Including other Templates);  当你发现一段内容反复的出现在你的子模板中，则意味着你需要将这个内容加入到你的父模板中，或者将这个内容提出来作为一个模板，然后用include标签 来包含这个模板

    * If you need to get the content of a block from the parent template, you can use the {{ parent() }} function. This is useful if you want to add to the contents of a parent block instead of completely overriding it: 如果你需要引入父模板中对应的block中的内容时，可以使用parent()方法进行输出,如下例

    {% block sidebar %}
        <h3>Table of Contents</h3>

            {# ... #}

        {{ parent() }}
    {% endblock %}

##模板的命名的位置

模板一般会存放在2个不同的地方

    * app/Resources/views/: The applications views directory can contain application-wide base templates (i.e. your application's layouts and templates of the application bundle) as well as templates that override third party bundle templates 该目录下存放的是项目的基础模板，或者是bundle的基础模板

    * path/to/bundle/Resources/views/: Each third party bundle houses its templates in its Resources/views/ directory (and subdirectories). When you plan to share your bundle, you should put the templates in the bundle instead of the app/ directory. 这个目录存放的是项目的子模板。

###在bundle中引入一个模板

对于在bundle中的模板，symfony使用的是 bundle:directory:filename的语法来访问模板

举例说明：AcmeBlogBundle:Blog:index.html.twig

    AcmeBlogBundle: (bundle) the template lives inside the AcmeBlogBundle  代表的是acmeBlogBundle,目录位置为src/Acme/BlogBundle/Resource/views目录

    Blog: (directory) indicates that the template lives inside the Blog subdirectory of Resources/views;Blog代表的是view目录下的Blog的目录

    index.html.twig: (filename) the actual name of the file is index.html.twig. 代表的是文件名

下例 AcmeBlogBundle::layout.html.twig
    
    layout.html.twig这个文件直接存在于src/Acme/BlogBundle/Resource/views目录下

###模板的后缀

不同的后缀代表不同的解析引擎,和格式化的方式

    Blog/index.html.twig   twig的引擎   html格式化方式
    Blog/index.html.php    php的引擎    html格式化方式
    Blog/index.css.twig    twig引擎     css的格式化方式

##Tags and Helpers
    
###Including other Templates

通过向包含的模板中传如不同的参数来实现展示内容的不同,如下例

一个包含变量的模板,其中有变量article
    
    {# app/Resources/views/Article/articleDetails.html.twig #}
    <h2>{{ article.title }}</h2>
    <h3 class="byline">by {{ article.authorName }}</h3>

    <p>
        {{ article.body }}
    </p>
    
一个需要引入article的模板

    {# app/Resources/views/Article/list.html.twig #}
    {% extends 'layout.html.twig' %}

    {% block body %}
        <h1>Recent Articles<h1>

            {% for article in articles %}
                    {{ include('Article/articleDetails.html.twig', { 'article': article }) }}
            {% endfor %}
    
    {% endblock %}

通过include一个模板并且传入一个article变量 ，在articleDetail.html.twig模板中是可以使用list.html.twig模板中定义的变量，如果在include()方法中设置with_context 为false时则不能使用外部的模板变量

###嵌入控制器

当你需要进行数据库操作或者一些复杂的逻辑操作的时候，一个模板就不足以完成任务，你需要在模板里面嵌入一个控制器已完成任务

一个可以嵌入的控制器
    
    // src/AppBundle/Controller/ArticleController.php
    namespace AppBundle\Controller;

    // ...

    class ArticleController extends Controller
    {
    public function recentArticlesAction($max = 3)
    {
        // make a database call or other logic
        // to get the "$max" most recent articles
        $articles = ...;

        return $this->render(
                'Article/recentList.html.twig',
                array('articles' => $articles)
                );
    }
    }

render的模板为
    
    {# app/Resources/views/Article/recentList.html.twig #}
    {% for article in articles %}
        <a href="/article/{{ article.slug }}">
                {{ article.title }}
        </a>
    {% endfor %}

可以嵌入一个控制器

    {# app/Resources/views/base.html.twig #}

    {# ... #}
    <div id="sidebar">
        {{ render(controller('AcmeArticleBundle:Article:recentArticles',{ 'max': 3 })) }}
    </div>

建议需要include时，直接引入一个控制器，而不要只引入一个模板

###Asynchronous Content with hinclude.js

    {{ render_hinclude(controller('...')) }}
    {{ render_hinclude(url('...')) }}

####出现问题
1.在页面中没有反应

####解决方法


##linking to pages

在模板中创建一个链接到另一个页面可以使用path(路由名，参数json数据) 

配置文件中的路由

    示例1
    app/config/routing.yml
    _welcome:
        path:     /
            defaults: { _controller: AppBundle:Welcome:index }

    示例2
    app/config/routing.yml
    article_show:
        path:     /article/{slug}
            defaults: { _controller: AppBundle:Article:show }
    


twig模板中的链接

    示例1

    <a href="{{ path('_welcome') }}">Home</a>

    示例2

    {# app/Resources/views/Article/recentList.html.twig #}
    {% for article in articles %}
        <a href="{{ path('article_show', {'slug': article.slug}) }}">
                {{ article.title }}
        </a>
    {% endfor %}

在模板中也可以用url()函数获取URL

    <a href="{{ url('_welcome') }}">Home</a>

##linking to Assets

可以使用asset的函数

    <img src="{{ asset('images/logo.png') }}" alt="Symfony!" />

    <link href="{{ asset('css/blog.css') }}" rel="stylesheet" type="text/css" />

    可以设置版本号
    
    <img src="{{ asset('images/logo.png', version='3.0') }}" alt="Symfony!" />

    可以设置显示为绝对目录

    <img src="{{ asset('images/logo.png', absolute=true) }}" alt="Symfony!" />

##在模板中包含样式文件和js文件

    {# app/Resources/views/base.html.twig #}
    <html>
    <head>
    {# ... #}

    {% block stylesheets %}
    <link href="{{ asset('css/main.css') }}" rel="stylesheet" />
    {% endblock %}
    </head>
    <body>
    {# ... #}

    {% block javascripts %}
    <script src="{{ asset('js/main.js') }}"></script>
    {% endblock %}
    </body>
    </html>

##Global Template Variables
全局的模板变量
    
    * app.security - The security context.
    
    * app.user - The current user object.

    * app.request - The request object.

    * app.session - The session object.

    * app.environment - The current environment (dev, prod, etc).

    * app.debug - True if in debug mode. False otherwise.


    <p>Username: {{ app.user.username }}</p>
    {% if app.debug %}
        <p>Request method: {{ app.request.method }}</p>
        <p>Application Environment: {{ app.environment }}</p>
    {% endif %}

##配置和使用模板服务

    return $this->render('Article/index.html.twig');

    直接调用render()方法相当于

    use Symfony\Component\HttpFoundation\Response;

    $engine = $this->container->get('templating');
    $content = $engine->render('Article/index.html.twig');

    return $response = new Response($content);

先获取模板引擎，在加载模板，在返回一个response、

可以在配置文件中对模板引擎进行配置
    
    app/config/config.yml
    framework:
         ...
            templating: { engines: ['twig'] }

    
##覆盖模板文件

当你下载了第三方bundle的时候，会需要覆盖它的默认的模板文件，因为symfony会去2个地方寻找模板

示例
    
    1. app/Resources/AcmeBlogBundle/views/Blog/index.html.twig

    2. src/Acme/BlogBundle/Resources/views/Blog/index.html.twig

这个时候在app的目录下的模板文件会覆盖掉src目录下的模板文件


##三个等级的继承

    * Create a app/Resources/views/base.html.twig file that contains the main layout for your application (like in the previous example). Internally, this template is called base.html.twig;第一步创建一个基础模板

    * Create a template for each "section" of your site. For example, the blog functionality would have a template called Blog/layout.html.twig that contains only blog section-specific elements;第二步创建一个站点模板，它继承基础模板

    * Create individual templates for each page and make each extend the appropriate section template. For example, the "index" page would be called something close to Blog/index.html.twig and list the actual blog posts.创建单独的页面模板，他继承站点模板

##output escaping

    格式化输出，可以通过filter实现

    {{ article.body|raw }}

##可以对模板进行语法检查

     You can check by filename:
    $ php app/console twig:lint app/Resources/views/Article/recentList.html.twig

     or by directory:
    $ php app/console twig:lint app/Resources/views

##模板的格式化

如展示一个XML模板

    * XML template name: Article/index.xml.twig
    
    * XML template filename: index.xml.twig

    public function indexAction(Request $request)
    {
            $format = $request->getRequestFormat();

                return $this->render('Blog/index.'.$format.'.twig');
    }

在twig中创建一个pdf的链接

    <a href="{{ path('article_show', {'id': 123, '_format': 'pdf'}) }}">
        PDF Version
    </a>

By anni @Global City 2014-12-12













    
























    
