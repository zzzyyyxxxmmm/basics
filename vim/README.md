# 常见操作
1. 删除所有不带数字的
```
:v/\d/d
```

2. 交换括号里的参数
```
:%s/(\(.*\), \(.*\))/(\2, \1)<CR>ZZ

多个需要交换
qq%db%p dt, pq<CR>@qZZ
```



```vim
gUaw
~
vU
vu
c3w
```



<div align=center>
<img src="https://github.com/zzzyyyxxxmmm/basics/blob/master/image/vim_1.png" width="500" height="300">
</div>

# Repeatable Actions and How to Reverse Them
<div align=center>
<img src="https://github.com/zzzyyyxxxmmm/basics/blob/master/image/vim_2.png" width="500" height="300">
</div>

# replace with cw
<div align=center>
<img src="https://github.com/zzzyyyxxxmmm/basics/blob/master/image/vim_3.png" width="500" height="300">
</div>

u恢复的是
```
i{insert some text}<Esc>
```
所以经常切换到normal mode可以control the granularity

### Use a Count When It Matters
<div align=center>
<img src="https://github.com/zzzyyyxxxmmm/basics/blob/master/image/vim_4.png" width="500" height="300">
</div>

### Make Corrections Instantly from Insert Mode
<div align=center>
<img src="https://github.com/zzzyyyxxxmmm/basics/blob/master/image/vim_5.png" width="500" height="300">
</div>

### Paste from a Register Without Leaving Insert Mode
<div align=center>
<img src="https://github.com/zzzyyyxxxmmm/basics/blob/master/image/vim_6.png" width="500" height="300">
</div>

### Toggling the Free End of a Selection
<div align=center>
<img src="https://github.com/zzzyyyxxxmmm/basics/blob/master/image/vim_7.png" width="500" height="300">
</div>

### Edit Tabular Data with Visual-Block Mode
<div align=center>
<img src="https://github.com/zzzyyyxxxmmm/basics/blob/master/image/vim_8.png" width="300" height="800">
</div>

### Duplicate or Move Lines Using ‘:t’ and ‘:m’ Commands
<div align=center>
<img src="https://github.com/zzzyyyxxxmmm/basics/blob/master/image/vim_9.png" width="800" height="300">
</div>

<div align=center>
<img src="https://github.com/zzzyyyxxxmmm/basics/blob/master/image/vim_10.png" width="800" height="300">
</div>

<div align=center>
<img src="https://github.com/zzzyyyxxxmmm/basics/blob/master/image/vim_11.png" width="800" height="300">
</div>

<div align=center>
<img src="https://github.com/zzzyyyxxxmmm/basics/blob/master/image/vim_12.png" width="800" height="300">
</div>

### Run Normal Mode Commands Across a Range
```:%normal A;```

### Trace Your Selection with Precision Text Objects
<div align=center>
<img src="https://github.com/zzzyyyxxxmmm/basics/blob/master/image/vim_13.png" width="300" height="800">
</div>
或者用vi

### Delete Around, or Change Inside
用ciw 代替dawi

### global mark
```
mm
'm 跳转到标记行
`m 跳转到标记点
```
### Delete, Yank, and Put with Vim’s Unnamed Register
```
xp
Xp
xP
ddp
yyp
```

y默认放在""和"0的register, 因此d,x是不影响y的register的
### Grok Vim’s Registers
<div align=center>
<img src="https://github.com/zzzyyyxxxmmm/basics/blob/master/image/vim_14.png" width="500" height="300">
</div>

<div align=center>
<img src="https://github.com/zzzyyyxxxmmm/basics/blob/master/image/vim_15.png" width="500" height="300"> 
</div>

<div align=center>
<img src="https://github.com/zzzyyyxxxmmm/basics/blob/master/image/vim_16.png" width="500" height="300">
</div>

<div align=center>
<img src="https://github.com/zzzyyyxxxmmm/basics/blob/master/image/vim_17.png" width="800" height="500">
</div>

gp真的很好用!!!

### Record and Execute a Macro
```
qa
A;[ESC]
q
@a
@@
@@
100@a
:normal @a
```

### Recording and Playing a Macro with a Count
<div align=center>
<img src="https://github.com/zzzyyyxxxmmm/basics/blob/master/image/vim_18.png" width="800" height="300">
</div>

### Evaluate an Iterator to Number Items in a List
```
:let i=1
qa
<Esc>
:let i+=1
q
jVG
:'<,'>normal @a

qa
H
f 
vHx
q
:'<,'>normal @a
```


# Vim练习

## 1
start file:
```
This is a
very short

file, but it is 
still
full

of

surpises.
```

end file:
```
*This is a
*very short

*file, but it is 
*still
*full

*of

*surpises.
```


利用g匹配找到所有非空行, 然后加*
```sh
:g/^./norm I*<CR>ZZ

:g/./norm I*<CR>ZZ

:g/./s/^/*<CR>ZZ 13

:%s/./*&<CR>ZZ 11

<C-V>GyPgvr*ZZ 10
```

## 2
**start file:**
```
super.onCreate(savedInstanceState)
setContentView(R.layout.activity_second)
Intent intent = getIntent()
String text = intent.getStringExtra("text")

TextView view = findViewById(R.id.textView2)
view.setText(text)
```

**end file:**
```
super.onCreate(savedInstanceState);
setContentView(R.layout.activity_second);
Intent intent = getIntent();
String text = intent.getStringExtra("text");

TextView view = findViewById(R.id.textView2);
view.setText(text);
```

**Solution:**
```
:%s/.$/&;<CR>ZZ
```

## 3
```
6GYp<C-A>ws11<Esc><fd-61>3joNew text.<CR><Esc>ZZ

MYp<C-A>l11<C-A>GONew te<C-N>.<CR><Esc>ZZ
```

## 4
```
:%s/\(\d\)/-\1<CR>ZZ

:%s/\d/-&<CR>ZZ
```

## 5
```
i# <Esc>fiswa<Esc>A #<Esc>YPPVr#G.ZZ
```

## 6
```
//先替换再删除所有空行
dj:%s/,/\r/g<CR>:g/^$/d<CR>ZZ

dj:%s/,/\r/g<CR>:v/./d<CR>ZZ


dj3JIwr<CR><Esc>u9@.$xZZ

dj3JV"=[<C-R><C-L>]<CR>pZZ
```

## 7
question:
```
I can't see the $ for all the $,
But there $ not $ $ tomorrow.
$ she can do $ and 4,
$ the $ in the $ and $ the $.

----------------------------------------

I can't see the 1 for all the 2,
But there 1 not 2 3 tomorrow.
1 she can do 2 and 4,
1 the 2 in the 3 and 4 the 5.
```

Solution:
```
/\$<CR>r1nr2nr1nr2nr3nr1nr2nr1nr2nr3nr4nr5ZZ

f$r1;r2+;r1;r2;r3+r1;r2+r1;r2;r3;r4;r5ZZ

:%s/\$/1<CR>:<Up><BS>2<CR>:<Up><BS>3<CR>:<Up><BS>4<CR>:<Up><BS>5<CR>ZZ

:let@a=@a+1|%s'\$'\=@a<CR>4@:ZZ

*dsa*
*dsadas*
*dsad*
```

## 8

```
foo = a
      ab
      abc
 
 
foo = "a"
      "ab"
      "abc"
 
 
 
:%s/\w\+$/"\0"/g<CR>ZZ
:%s/\w\+$/"&"/g<CR>ZZ
 
qqfai"<C-O>$"<Esc><Down>0q2@qZZ
 
```

### 9
```
## Headers

## To

## Underline


Headers
-------

Are
---

Underlined
----------

<C-V>GtUxqwYpVr-<CR><CR>qCAre<Esc>@wAd<Esc>@wZZ
```
 

 ### 10
```
Assert.ThrowsAsync<Exception>(() => _auction.StartSellingItem());
Assert.ThrowsAsync<Exception>(() => _application.StartBiddingIn(_auction));
Assert.ThrowsAsync<Exception>(() => _auction.HasReceivedJoinRequestFromSniper());
Assert.ThrowsAsync<Exception>(() => _auction.AnnounceClosed());
Assert.ThrowsAsync<Exception>(() => _application.ShowsSniperHasLostAuction());


_auction.StartSellingItem();
_application.StartBiddingIn(_auction);
_auction.HasReceivedJoinRequestFromSniper();
_auction.AnnounceClosed();
_application.ShowsSniperHasLostAuction();



qq2df $<BS>x<CR>q5@qZZ

:%norm d2W%x<CR>ZZ
```