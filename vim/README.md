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