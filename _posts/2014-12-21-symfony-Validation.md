---
layout: post
category: symfony
tags: [php , symfony]
---

#validation

####调整状态，在验证这一块重新开始

##The Basics of Validation

第一步在entity中声明一个类

    // src/Acme/BlogBundle/Entity/Author.php
    namespace Acme\BlogBundle\Entity;

    class Author
    {
            public $name;
    }

接下来对$name进行非空的定义，使用annotation的方法

    // src/Acme/BlogBundle/Entity/Author.php

    // ...
    use Symfony\Component\Validator\Constraints as Assert;

    class Author
    {
        /**
         * @Assert\NotBlank()
         */
        public $name;
    }

##Using the validator Service

定义好后可以在类中使用验证的服务进行验证

    // ...
    use Symfony\Component\HttpFoundation\Response;
    use Acme\BlogBundle\Entity\Author;

    public function indexAction()
    {
        $author = new Author();
        // ... do something to the $author object

        $validator = $this->get('validator');
        $errors = $validator->validate($author);

        if (count($errors) > 0) {
            /*
             * Uses a __toString method on the $errors variable which is a
             * ConstraintViolationList object. This gives us a nice string
             * for debugging
             */
            $errorsString = (string) $errors;

            return new Response($errorsString);
        }

        return new Response('The author is valid! Yes!');
    }

如果为空则返回错误信息

可以将错误信息传到一个模板中

    if (count($errors) > 0) {
            return $this->render('AcmeBlogBundle:Author:validate.html.twig', array(
                    'errors' => $errors,
                        ));
    }

这样可以遍历错误信息

    {# src/Acme/BlogBundle/Resources/views/Author/validate.html.twig #}
    <h3>The author has the following errors</h3>
    <ul>
    {% for error in errors %}
        <li>{{ error.message }}</li>
        {% endfor %}
        </ul>


##Validation and Forms

验证可以用在任何的类中，实际上在表单中使用validation的次数比较多

    // ...
    use Acme\BlogBundle\Entity\Author;
    use Acme\BlogBundle\Form\AuthorType;
    use Symfony\Component\HttpFoundation\Request;

    public function updateAction(Request $request)
    {
        $author = new Author();
        $form = $this->createForm(new AuthorType(), $author);

        $form->handleRequest($request);

        if ($form->isValid()) {
            // the validation passed, do something with the $author object

            return $this->redirect($this->generateUrl(...));
        }

        return $this->render('BlogBundle:Author:form.html.twig', array(
                    'form' => $form->createView(),
                    ));
    }

##配置

在配置文件中配置validation模块

    # app/config/config.yml
    framework:
        validation: { enable_annotations: true }

##Constraints(约束条件)

validation可以使用多个约束条件对应一个属性，validation有多种约束条件。

###Supported Constraints(支持的约束条件)(都为annotation写法)

####Basic Constraints(基础约束条件)

NotBlank:非空

   // src/Acme/BlogBundle/Entity/Author.php
   namespace Acme\BlogBundle\Entity;

   use Symfony\Component\Validator\Constraints as Assert;

   class Author
   {
       /**
        * @Assert\NotBlank()
        */
       protected $firstName;
   }

Blank: 为空

    // src/Acme/BlogBundle/Entity/Author.php
    namespace Acme\BlogBundle\Entity;

    use Symfony\Component\Validator\Constraints as Assert;

    class Author
    {
        /**
         * @Assert\Blank()
         */
        protected $firstName;
    }

NotNull:不为Null

    // src/Acme/BlogBundle/Entity/Author.php
    namespace Acme\BlogBundle\Entity;

    use Symfony\Component\Validator\Constraints as Assert;

    class Author
    {
        /**
         * @Assert\NotNull()
         */
        protected $firstName;
    }
Null : 为Null

    // src/Acme/BlogBundle/Entity/Author.php
    namespace Acme\BlogBundle\Entity;

    use Symfony\Component\Validator\Constraints as Assert;

    class Author
    {
        /**
         * @Assert\Null()
         */
        protected $firstName;
    }

True: 为真 在验证用户时，可以作用在一个方法上，判断其是否可以返回一个真值

    // src/Acme/BlogBundle/Entity/Author.php
    namespace Acme\BlogBundle\Entity;

    use Symfony\Component\Validator\Constraints as Assert;

    class Author
    {
        protected $token;

        /**
         * @Assert\True(message = "The token is invalid")
         */
        public function isTokenValid()
        {
            return $this->token == $this->generateToken();
        }
    }

false: 为假 可以在定义的时候定义一个返回的信息

    // src/Acme/BlogBundle/Entity/Author.php
    namespace Acme\BlogBundle\Entity;

    use Symfony\Component\Validator\Constraints as Assert;

    class Author
    {
        /**
         * @Assert\False(
         *     message = "You've entered an invalid state."
         * )
         */
        public function isStateInvalid()
        {
            // ...
        }
    }

type: 可以对类型进行限制  , 在返回的信息中可以使用变量{{value}} 和 {{type}}

    // src/Acme/BlogBundle/Entity/Author.php
    namespace Acme\BlogBundle\Entity;

    use Symfony\Component\Validator\Constraints as Assert;

    class Author
    {
        /**
         * @Assert\Type(type="integer", message="The value {{ value }} is not a valid {{ type }}.")
         */
        protected $age;
    }

    支持的类型有
    
    1. array(数组)

    2. string(字符串)

    3. bool(布尔值)

    4. callable(合法的可调用结构，字符串为函数，数组为对象)

    5. float(单精度浮点数)

    6. double（双精度浮点数）

    7. int(单精度整型)

    8. integer（单精度整型）

    9. long（整型）

    10. null（null）

    11. numeric(数字)

    12. object(对象)

    13. real（浮点数）

    14. resource(资源)

    15. scalar(标量)：标量就是字符串，整型，浮点型，布尔型

    16. string（字符串）

如果开启ctype的扩展就可以使用ctype的function:

ctype_alnum():判断是否是字母和数字
    
    <?php
    $strings = array('AbCd1zyZ9', 'foo!#$bar');
    foreach ($strings as $testcase) {
        if (ctype_alnum($testcase)) {
            echo "The string $testcase consists of all letters or digits.\n";
        } else {
            echo "The string $testcase does not consist of all letters or digits.\n";
        }
    }
    ?>

ctype_alpha():判断是否都是字母（包括大小写）

ctype_cntrl():判断是不是控制字符

ctype_digit():判断是不是数字，包含小数点不算

ctype_graph():判断是不是可打印的字符(不包括空格)

ctype_lower():判断是不是都是小写字符

ctype_print():判断是不是可打印的字符（包含空格）

ctype_punct():检测可打印的字符是不是不包含空白、数字和字母

ctype_space():做空白字符检测

ctype_upper():检查 string 和 text 里面的字符是不是都是大写字母。

ctype_xdigit():检测字符串是否只包含十六进制字符

####String Constraints(字符约束条件)

Email : 验证一个值是不是email地址

    // src/Acme/BlogBundle/Entity/Author.php
    namespace Acme\BlogBundle\Entity;

    use Symfony\Component\Validator\Constraints as Assert;

    class Author
    {
        /**
         * @Assert\Email(
         *     message = "The email '{{ value }}' is not a valid email.",
         *     checkMX = true
         * )
         */
        protected $email;
    }

    options:
    
    strict type: boolean default: false 是否为严格模式，为false时只是正则判断，为true时就需要加外接包 [egulias/email-validator](https://packagist.org/packages/egulias/email-validator) library 

    message: type: string default: This value is not a valid email address. 验证不通过时返回的信息

    checkMX：type: Boolean default: false  为真就调用checkdnsrr函数对host进行检查

    checkHost： type: Boolean default: false 为真就调用checkdnsrr函数对host进行检查

[Length](http://symfony.com/doc/current/reference/constraints/Length.html):检查字符的长度。

[URL](http://symfony.com/doc/current/reference/constraints/Url.html): 验证是不是一个url字符串

[Regex](http://symfony.com/doc/current/reference/constraints/Regex.html):用正则进行匹配验证

[ip](http://symfony.com/doc/current/reference/constraints/Ip.html): 验证一个ip

[Uuid](http://symfony.com/doc/current/reference/constraints/Uuid.html):唯一ID验证

[Range](http://symfony.com/doc/current/reference/constraints/Range.html):在2个值间做随机值判断

###By anni @Global city
