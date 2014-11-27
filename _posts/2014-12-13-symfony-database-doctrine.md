---
layout: post
category: symfony
tags: [php,symfony,database,doctrine]
---

#Doctrine

##Configuring the database

配置信息写在app/config/parameters.yml文件中

    parameters:
        database_driver:    pdo_mysql
        database_host:      localhost
        database_name:      test_project
        database_user:      root
        database_password:  password

在app/config/config.yml文件中使用配置变量
    
    doctrine:
        dbal:
            driver:   "%database_driver%"
            host:     "%database_host%"
            dbname:   "%database_name%"
            user:     "%database_user%"
            password: "%database_password%"

####使用doctrine时需要在php中安装PDO扩展

使用命令创建一个数据库
    
    $ php app/console doctrine:database:create 
    
    可以在数据库中创建一个config中定义的database

删除一个数据库

    $ php app/console doctrine:database:drop --force

在配置文件中设定数据库的字符集
    
    doctrine:
        dbal:
            driver:   "%database_driver%"
            host:     "%database_host%"
            port:     "%database_port%"
            dbname:   "%database_name%"
            user:     "%database_user%"
            password: "%database_password%"
            charset:  UTF8

##创建一个Entity的类

    // src/AppBundle/Entity/Product.php
    namespace AppBundle\Entity;

    class Product
    {
            protected $name;
            protected $price;
            protected $description;
    }

一个entity类对应的一个表一个属性对应一个字段

###可以通过命令行来创建Entity

    php app/console doctrine:generate:entity

生成映射的方式可以写yml文件也可以用annotation等方式

在生成时可以选择生成一个继承Repository类的子类，进行重写该表的处理方法
    
####产生的问题

1. 官方文档中说在一个bundle中是不能用2种方式来描述entity的



####解决方法

1.


#####[doctrine的基础的映射文档](http://docs.doctrine-project.org/projects/doctrine-orm/en/latest/reference/basic-mapping.html)

#####tips:定义属性名称的时候不要和sql中保留关键字起冲突

###创建一个entity类中的get和set方法

    php app/console doctrine:generate:entities AppBundle/Entity/Product

####产生问题

1. 产生一个Namespace "AcmeDemoBundle\Entity\second" does not contain any mapped entities.的错误

####解决方法

1. 将命令改为php app/console doctrine:generate:entities Acme/DemoBundle/Entity/third  ，在entity类中class上的注释中加上 @ORM\Entity

###使用doctrine:generate:entities这个命令可以
    
    * generate getters and setters;

    * generate repository classes configured with the @ORM\Entity(repositoryClass="...") annotation;

    * generate the appropriate constructor for 1:n and n:m relations.

在执行命令后会生成一个带～的备份,可以在命令后加上  --no-backup 来取消备份

也可以为一个命名空间下所有entity类进行生成操作  app/console doctrine:generate:entities Acme 

##创建一个数据表

根据entity类生成数据表

    $ php app/console doctrine:schema:update --dump-sql 打印需要执行的sql语句

    $ php app/console doctrine:schema:update --force 执行数据表的生成


####产生的问题

1. Unknown database type enum requested, Doctrine\DBAL\Platforms\MySqlPlatform may not support it.

####解决方法

1.因为是第一次执行该命令，而且用的是mysql数据库，因为不支持mysql数据库中enum类型 可以在config/config.yml文件中加入配置

    doctrine:
        dbal:
            driver:   "%database_driver%"
            host:     "%database_host%"
            port:     "%database_port%"
            dbname:   "%database_name%"
            user:     "%database_user%"
            password: "%database_password%"
            charset:  UTF8
            mapping_types:
                enum: string

##创建一个对象插入到数据库中

    use AppBundle\Entity\Product;
    use Symfony\Component\HttpFoundation\Response;

    // ...
    public function createAction()
    {
        $product = new Product();
        $product->setName('A Foo Bar');
        $product->setPrice('19.99');
        $product->setDescription('Lorem ipsum dolor');

        $em = $this->getDoctrine()->getManager();

        $em->persist($product);
        $em->flush();

        return new Response('Created product id '.$product->getId());
    }

创建一个entity对象，进行属性设置，获取$em,使用persist()方法将entity对象添加进实体，在用flush()方法执行数据库操作（在实际应用中，建议进行多次persist后在进行flush 已减低数据库连接压力）

##从数据库中获取一个对象

通过Repository类中的方法进行查询操作

    public function showAction($id)
    {
        $product = $this->getDoctrine()
            ->getRepository('AppBundle:Product')
            ->find($id);

        if (!$product) {
            throw $this->createNotFoundException(
                    'No product found for id '.$id
                    );
        }

        // ... do something, like pass the $product object into a template
    }

Repository类中的其他方法

    // query by the primary key (usually "id")
    $product = $repository->find($id);

    // dynamic method names to find based on a column value
    $product = $repository->findOneById($id);
    $product = $repository->findOneByName('foo');

    // find *all* products
    $products = $repository->findAll();

    // find a group of products based on an arbitrary column value利用__call()方法实现的
    $products = $repository->findByPrice(19.99);                    

    // query for one product matching by name and price
    $product = $repository->findOneBy(
        array('name' => 'foo', 'price' => 19.99)
        );

    // query for all products matching the name, ordered by price
    $products = $repository->findBy(
        array('name' => 'foo'),
            array('price' => 'ASC')
            );

##更新一个对象

    public function updateAction($id)
    {
        $em = $this->getDoctrine()->getManager();
        $product = $em->getRepository('AppBundle:Product')->find($id);

        if (!$product) {
            throw $this->createNotFoundException(
                    'No product found for id '.$id
                    );
        }

        $product->setName('New product name!');
        $em->flush();

        return $this->redirect($this->generateUrl('homepage'));
    }

先获取一个对象，在进行更新操作 在进行flush 操作

##删除一个对象

    $em->remove($product);
    $em->flush();

##获取单个对象

    $repository->find($id);

    $repository->findOneByName('Foo');

##通过拼接请求获取对象

    $repository = $this->getDoctrine()
        ->getRepository('AppBundle:Product');

        $query = $repository->createQueryBuilder('p')
            ->where('p.price > :price')
             ->setParameter('price', '19.99')
             ->orderBy('p.price', 'ASC')
             ->getQuery();

        $products = $query->getResult();


#####也可以使用 
    
    $repository = $this->getDoctrine()->getEntityManager->getRepository("App\\DemoBundle\\Entity\\second");

getResult()会返回一个包含对象的数组

如果需要返回一个对象可以使用
    
    $product = $query->getOneOrNullResult()

##通过DQL获取对象

    $em = $this->getDoctrine()->getManager();
    $query = $em->createQuery(
            'SELECT p
            FROM AppBundle:Product p
            WHERE p.price > :price
            ORDER BY p.price ASC'
            )->setParameter('price', '19.99');

    $products = $query->getResult();

##Custom Repository Classes自定义库类

用户可以给每个entity自定义一个继承与Repository的自定义Repository

    // src/AppBundle/Entity/Product.php
    namespace AppBundle\Entity;

    use Doctrine\ORM\Mapping as ORM;

    /**
     * @ORM\Entity(repositoryClass="AppBundle\Entity\ProductRepository")
      */
      class Product
      {
              public function findAllOrderedByName()
              {
                  return $this->getEntityManager()
                      ->createQuery('SELECT p FROM symBundle:Register p ORDER BY p.name ASC')
                      ->getResult();
              }
      }

现在可以使用自定义的方法

    $em = $this->getDoctrine()->getManager();
    $products = $em->getRepository('AppBundle:Product')
                ->findAllOrderedByName();

##Entity Relationships/Associations 实体关系/协会

    $ php app/console doctrine:generate:entity --entity="AppBundle:Category" --fields="name:string(255)"

##Relationship Mapping Metadata 属性与属性间的关系映射

创建2个entity之间的联系（如创建一个关于category实体和product实体的联系）

在一个category的Entity中创建一个products属性

    // src/AppBundle/Entity/Category.php

    // ...
    use Doctrine\Common\Collections\ArrayCollection;

    class Category
    {
        // ...

        /**
         * @ORM\OneToMany(targetEntity="Product", mappedBy="category")
         */
        protected $products;

        public function __construct()
        {
            $this->products = new ArrayCollection();
        }
    }

注明是一个category对应多个product的关系

每一个product对象都对应一个category对象，在product中加入$category

    // src/AppBundle/Entity/Product.php

    // ...
    class Product
    {
        // ...

        /**
         * @ORM\ManyToOne(targetEntity="Category", inversedBy="products")
         * @ORM\JoinColumn(name="category_id", referencedColumnName="id")
         */
        protected $category;
    }

设置完成后在通过以下命令直接生成get和set命令

    php app/console doctrine:generate:entities Acme

生成get和set命令

在通过命令更新数据表生成外键

###Saving Related Entities

有联系的Entity的插入和保存

    use AppBundle\Entity\Category;
    use AppBundle\Entity\Product;
    use Symfony\Component\HttpFoundation\Response;

    class DefaultController extends Controller
    {
        public function createProductAction()
        {
            $category = new Category();
            $category->setName('Main Products');

            $product = new Product();
            $product->setName('Foo');
            $product->setPrice(19.99);
            $product->setDescription('Lorem ipsum dolor');
            // relate this product to the category
            $product->setCategory($category);

            $em = $this->getDoctrine()->getManager();
            $em->persist($category);
            $em->persist($product);
            $em->flush();

            return new Response(
                    'Created product id: '.$product->getId()
                    .' and category id: '.$category->getId()
                    );
        }
    }

上例中增加product时，需要将category的对象作为参数传入

###Fetching Related Objects

查询关联的entity

    public function showAction($id)
    {
        $product = $this->getDoctrine()
            ->getRepository('AppBundle:Product')
            ->find($id);

        $categoryName = $product->getCategory()->getName();

        // ...
    }






















