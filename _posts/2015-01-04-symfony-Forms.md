---
layout: post
category: symfony
tags: [php , symfony]
---

##Forms

处理表单在所有的web编程中使用最多的情况，下面开始symfony的form之旅,symfony使用外部的lib包进行管理

###Create simple form

    // src/AppBundle/Entity/Task.php
	namespace AppBundle\Entity;

	class Task
	{
	    protected $task;
	    protected $dueDate;
		
	    public function getTask()
	    {
	        return $this->task;
	    }
	
	    public function setTask($task)
	    {
	        $this->task = $task;
	    }

	    public function getDueDate()
	    {
       		 return $this->dueDate;
  	  }

   	 public function setDueDate(\DateTime $dueDate = null)
	    {
   	     $this->dueDate = $dueDate;
 	   }
	}

在symfony中是使用entity的方式进行表单管理，通常创建在entity的文件夹中










