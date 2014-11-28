---
title: git add -i
layout: default
excerpt: <p>Submitting meaningful commits to your repository makes it easier for your future self and team-mates from understanding how and why changes were 
         brought in. Instead of one gigantic commit that includes all the changes for a feature, splitting up your commits into a single commit per 
         feature allows you to look at meaningful diffs at a commit level. This makes code reviews go smoother as you won't have to remember why you
         made a change.</p>
 
---
### git add -i

Submitting meaningful commits to your repository makes it easier for your future self and team-mates in understanding how and why changes were 
brought in. Instead of one gigantic commit that includes all the changes for a feature, splitting up your commits into a single commit per 
feature allows you to look at meaningful diffs at a commit level. This makes code reviews go smoother as you won't have to remember why you
made a change.

Git has the excellent interactive feature which allows you to select a set of changes called a hunk to include in a commit. Git will allow 
you to split the hunk into smaller pieces to give you a degree of control over which changes to include. However splitting hunks has its limits.
To get really picky about what goes into the commit, get into patch mode. This allows you line by line control.

 {% highlight diff %} 
     diff --git a/LinkedList.php b/LinkedList.php  
     index b7d378f..5fcfce5 100644  
     --- a/LinkedList.php  
     +++ b/LinkedList.php  
     @@ -1,19 +1,24 @@  
      <?php  
      class LinkedList  
      {  
        private $nodes;  
       
        public function __construct($name)  
        {  
       
        }  
       
        public function addNode(Node $node)  
        {  
     -    $this->node[] = $node;  
     +    $this->node[] = $node;   
     +  }  
     +  
     +  public function traverseList($callback)  
     +  {  
     +    // Traverse the list and call the callback  
        }  
       
      }
{% endhighlight %}

Entering patch mode gives us the following screen:

{% highlight bash %}  
    # Manual hunk edit mode -- see bottom for a quick guide  
    @@ -13,7 +13,12 @@ class LinkedList  
      
       public function addNode(Node $node)  
       {  
    -    $this->node[] = $node;  
    +    $this->node[] = $node;   
    +  }  
    +  
    +  public function traverseList($callback)  
    +  {  
    +    // Traverse the list and call the callback  
       }  
      
     }  
    # ---  
    # To remove '-' lines, make them ' ' lines (context).  
    # To remove '+' lines, delete them.  
    # Lines starting with # will be removed.  
    #  
    # If the patch applies cleanly, the edited hunk will immediately be  
    # marked for staging. If it does not apply cleanly, you will be given  
    # an opportunity to edit again. If all lines of the hunk are removed,  
    # then the edit is aborted and the hunk is left unchanged.
{% endhighlight %}