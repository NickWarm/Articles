# block、proc、yield、lambda之間的關係

通常我們要定義一個method時會寫

```Ruby
def say_hello
  puts 'hello'
end
```

>Functions stand alone and methods are members of a class.
>
>A method is a piece of code that is called by a name that is associated with an object.
>
>ref：
>- [我們寫的是 Function 還是 Method ?](http://kurtyu1209.blogspot.tw/2011/10/function-method.html)
>- [oop - Difference between a method and a function - Stack Overflow](http://stackoverflow.com/questions/155609/difference-between-a-method-and-a-function)

有時，我們會想要給method傳參數進去

```Ruby
def multiply(x)
  puts x * 2
end
```

```
pry(main)> multiply(2)
=> 4
```

# proc 與 block

當傳參數已經無法滿足我了，我想要讓method使用一段程式碼(`block`)時，**method的參數加上`&`**。

```Ruby
def myfun(&p)
  p.call
end
```

>`block`是可以暫存一段ruby code的地方，如果只有一行可以用`{...}`，若是很多行可以用`do...end`
>
>`block`不是物件(object)，沒有辦法單獨的存在，也沒辦法把它指定給某個變數
>
>`block`也不是參數(variable)，`block`通常得像寄生蟲一樣依附或寄生在其它的方法或物件（或是使用某些類別把它物件化）
>
>ref
>- [第 07 章 - 方法與程式碼區塊(block) - iT 邦幫忙](http://railsbook.tw/chapters/07-ruby-basic-3.html)

`&`就像是一個聲明，說得是我要傳一段`block`進來，然後我用`call`調用傳進來的`block`

如此一來，我就能在我的method後面，接一個`block`，**把這個`block`給物件化**。

如此一來，method裡的`p`就是一個物件，可以調用`call`這方法，來取得`block`的內容

```
pry(main)> myfun { puts "Hello world" }
=> Hello world

pry(main)> myfun { puts 2*2 }
=> 4
```

當參數使用`&`，如果直接呼叫`myfun`後面沒給`block`就會噴錯，因為`myfun`裡的`call`是一個method，method的存在是因為有著一個object來呼叫它。


```
pry(main)> myfun()
=> NoMethodError: undefined method `call' for nil:NilClass

pry(main)> myfun
=> NoMethodError: undefined method `call' for nil:NilClass

pry(main)> myfun() {}
=> nil
```

## proc的使用時機

但是`block`的缺點是，他沒辦法把這段程式碼存起來，只有要用時放在method後面。

如果有 **想要重複使用的程式碼，就用`Proc`包起來**，`Proc`也就是Procedure的意思。

使用`Proc`就可以把`block`物件化

```Ruby
# 原本用block的寫法
myfun { puts "use myproc" }

# 改用Proc寫

myproc = Proc.new { puts "use myproc" }

# 等同於

myproc = proc { puts "use myproc" }
```

**`Proc`的default method就是`call`**，讓我們能呼叫要重複使用的程式碼
- [Class: Proc (Ruby 2.4.0) - call](https://ruby-doc.org/core-2.4.0/Proc.html#method-i-call)

```

[40] pry(main)> myproc2 = proc { |x| x *2 }
=> #<Proc:0x007fde7c666d60@(pry):66>

[41] pry(main)> myproc2.call(2)
=> 4

[42] pry(main)> myproc2.call(3)
=> 6
```

## 惱人的`&`

當method的參數使用`&`，後面又接`Proc`時

```Ruby
# method
def myfun(&p)
  p.call
end

myproc = proc { puts "use myproc" }
```

執行程式時

```
pry(main)> myfun(&myproc)
=> use myproc

# 由於Ruby可以省略括號，所以也可以寫成

pry(main)> myfun &myproc
=> use myproc

# 由於我們在myfun這method有用 & 來定義進來的參數，所以myfun這method後面只能接block，
# 如果 myfun 後面接 Proc 生成的 object，則會噴錯

pry(main)> myfun myproc
=> ArgumentError: wrong number of arguments (1 for 0)
```



如果說，我在method的參數，不用`&`則會怎樣呢？

```Ruby
def myfun2(p)
  p.call
end

myproc = Proc.new { puts "use myproc" }

pry(main)> myfun2 myproc   # 這就是一般的 Proc 調用 call 方法
=> use myproc


pry(main)> myfun2 &myproc  # 使用 & 就是在聲明使用 block，若是使用block，則 myfun2的參數就該定義 &，由於myfun2的參數沒定義 & 所以噴錯
=> ArgumentError: wrong number of arguments (0 for 1)
```

如果自定義的method，**它的參數不用`&`，而method裡又使用了`call`**。則必定要傳`Proc`進去。因為`call`是`Proc`物件的**default method**。

>一個重要的小結論：
>
>當method的定義裡面有用到`call`時，method的參數(變數)`p`
>- 加上`&`：`&p`代表method後面要接則`block`，透過`&p`使`block`物件化，當`p.call`調用時，就會執行`block`。
>- 沒有`&`：`p`代表要傳`proc`這個object進到method裡去，當`p.call`調用時，就會執行傳進來的`proc`。

# yield

當我們想要傳`block`進入method時，會寫

```Ruby
def myfun(&p)
  p.call
end
```

如果想要省略掉`&`與`call`，則我們可以使用`yield`，寫成

```Ruby
def myfun3
  yield
end
```

換句話說

```Ruby
# 原始寫法

def myfun(&p)
  p.call
end

# 等同於

def myfun3
  yield
end

```

使用`yield`：

```Ruby
# 能傳block進來
pry(main)> myfun3 { puts "Hello world" }
=> Hello world

# 傳proc就會噴錯
myproc = Proc.new { puts "use myproc" }

pry(main)>myfun3 myproc
=> ArgumentError: wrong number of arguments (1 for 0)
```

>小結：`yield` = 方便執行block的方式。`yield`語句同等於省略`&block`輸入 以及 `block.call`

# lambda

要讓method使用一段程式碼，除了用`block`或`Proc`之外，也能用`lambda`。

`lambda`的寫法很簡單，例如：

```Ruby
lambda1 = lambda {|x| x * 2}

# 等同於

lambda2 = ->(x) { return x * 2 }

# 等同於

lambda3 = ->(x) { x * 2 }  # Ruby可以省略return
```

我們先看一下`lambda`與`Proc`的關係

```
pry(main)> lambda2.class
=> Proc
```

可以看到，**`lambda`的class就是`Proc`**。由於`lambda`是`Proc`，所以`lambda`能調用`Proc`的default method也就是`call`。

```
pry(main)> lambda1.call("hello world")
=> "hello worldhello world"

pry(main)> lambda2.call("hello world")
=> "hello worldhello world"

pry(main)> lambda3.call("hello world")
=> "hello worldhello world"
```

# proc與lambda的差異

`lambda`和`proc`的主要差別有兩個

1. `lambda`會check參數數量，`proc`不會
2. `return`的處理不同，lambda的`return`只會跳出lambda，**proc的`return`會跳出整個method**

## `lambda`會check參數數量，`proc`不會

假如我定義`lambda`時只有一個參數，然後丟兩個參數進去就會噴錯

```
[3] pry(main)> lam2 = ->(x) {puts  x*2}
=> #<Proc:0x007fde7c5d6c60@(pry):3 (lambda)>

[4] pry(main)> lam2.call(3)
6
=> nil

[5] pry(main)> lam2.call(3,4)
ArgumentError: wrong number of arguments (2 for 1)
```

但若是`Proc`，只定義一個參數丟兩個參數進去，則會只取第一個參數

```
[6] pry(main)> proc1 = proc {|x| puts x * 2}
=> #<Proc:0x007fde7c54b610@(pry):6>

[7] pry(main)> proc1.call(2,3,4)
4
=> nil
```

換句話說，**`lambda`對參數的確認較為嚴謹**

## `return`的處理不同

lambda的`return`只會跳出lambda

```Ruby
#proc and lambda
def run_a_proc(p)
  puts 'start...'
  p.call
  puts 'end.'
end

#the lambda will be ignore
def run_couple
  run_a_proc lambda { puts 'I am a lambda'; return }
  run_a_proc proc { puts 'I am a proc'; return }
end

pry(main)> run_couple
=> start...
=> I am a lambda
=> end.
=> start...
=> I am a proc
=> nil


```

proc的`return`會跳出整個method

```Ruby
#proc and lambda
def run_a_proc(p)
  puts 'start...'
  p.call
  puts 'end.'
end

#the lambda will be ignore
def run_couple2
  run_a_proc proc { puts 'I am a proc'; return }
  run_a_proc lambda { puts 'I am a lambda'; return }
end

pry(main)> run_couple2
=> start...
=> I am a proc
=> nil
```

# block、Proc、lambda的小整理

```Ruby
# 使用yield取代 & 與 call
def f1
  yield
end

f1 { puts "f1" }


# 使用block
def f2(&p)
  p.call
end

f2 { puts "f2" }


# 使用proc與lambda
def f3(p)
  p.call
end

f3(proc{ puts "f3"})

f3(lambda{puts "f3"})
```

---

ref
- [Rails 高級新手系列 - 關於Ruby « Wayne](http://waynechu.logdown.com/posts/200843-about-ruby)
- [Ruby 中的 block, Proc 和 Lambda ~ 葉學舫 | HFYEH](http://sharefun010407.blogspot.tw/2016/11/ruby-block-proc-lambda.html)
- [聊聊 Ruby 中的 block, proc 和 lambda · Ruby China](https://ruby-china.org/topics/10414)
- [method / block / yield / Proc / lambda 全面解釋 - 教學 - Rails Fun!! Ruby & Rails 中文論壇](http://railsfun.tw/t/method-block-yield-proc-lambda/110)
- [Ruby Block的變形與使用 - Ruby - Rails Fun!! Ruby & Rails 中文論壇](http://railsfun.tw/t/ruby-block/54)
- [Block & Proc/Lambda « Brian 's Blog](http://brian-p-pan-blog.logdown.com/posts/195007-ruby-block)
- [Ruby Proc - Lamdba | Yield - JT's Blog](http://rt-tong.com/blog/2016/08/10/ruby-proc-lamdba-yield/)
- [Ruby Method - \*argv | &block - JT's Blog](http://rt-tong.com/blog/2016/08/05/ruby-method/)
- [理解 Ruby 中的 Blocks，Procs 和 Lambdas – Jason's blog](http://liuzxc.github.io/articles/ruby-block-proc-lambdas/)
- [程式區塊與 Proc - openhome.cc](https://openhome.cc/Gossip/Ruby/Proc.html)
- [Ruby中Proc，Lambda - 简书](http://www.jianshu.com/p/5f205f650a90)
