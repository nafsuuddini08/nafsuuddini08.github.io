---
layout: single
title: Tmux Shortcuts
excerpt: "Tmux is a tool that allows us to divide a terminal in several panels and also to speed up when we are working in the console with different functions and shortcuts."
date: 2021-11-25
classes: wide
header:
  teaser: /assets/images/tmux/tmux22.png
  teaser_home_page: true
  icon: /assets/images/
categories:
  - zsh
tags:
  - tmux
  - linux
---

<p align = "center">
<img src = "/assets/images/tmux/tmux22.png">
</p>

Tmux is a tool that allows us to divide a terminal in several panels and also to speed up when we are working in the console with different functions and shortcuts.

## Tmux shortcuts

prefix

```
ctrl + b 
```
close session

```
tmux new -s "nameofthesession"  
```
change session name

```
prefix + $  
```
list session

```
tmux ls 
```
kill session

```
tmux kill-session -t [name]  
```
enter a session

```
tmux attach -t [name] 
```
rename window

```  
ctrl + b + , 
```
create a window

```
ctrl + b + c  
```
move from windows

```
ctrl + b + 1,2,3,4  
```
vertical window

```
ctrl + b + shift + 2 
```
window in horizontal position

```
ctrl + b + shift + 5   
```
move the second panel in horizontal or in vertical.

```
ctrl + b + space
```
see the hour in the panel.

```
prefix + t 
```
move in terminal to write 

```
prefix + arrows - up - down  
```

delete bash o zsh window

```
prefix + & 
```

resize the windows, (hold down the ctrl key) 

```
prefix + arrows 
```
see the modification of a file in another window in real time, like a live serve.

```
tail -f namefile 
```

allows us to see the sessions that we have created in tmux and windows that we have created, and we can see what we have done in that session or sale previously.

```
ctrl + a + s 
```
another way to view sessions with hierarchy. 

```
prefix + w 
```
to see the number of panels in the terminal.

```
prefix + q  
```
delete or kill sessions.

```
ctrl + a + s + enter 
```
allows you to move the content of a panel from bottom to top, right or left.

```
prefix + {} 
```
permite mover los contenidos de los paneles en el sentido de agujas del reloj.

```
prefix + o  
```
If we have made a mistake in putting a command in the window that we don't want, we can move to another new window and do "prefix + shift 1" to move the other content in the correct window. 

```
prefix + shift + 1 
```
use the mouse to size the panels and move windows. To deactivate this option, execute the same shurcut and enter again. 

```
prefix + m
```

unify or compact windows with = ***"join-pane -s (window we are currently in) 1 -t (and in the window we want to move) 2"***

```
prefix + shift + :
```

copy the content of a file. 

```
prefix + [ (copy mode), ctrl + espacio (selection mode), alt + w (copy content), prefix + ] (paste content in the clipboard)   
```
search for info using filters.

```
prefix + [ (copy mode) + ctrl + s  
```
Do window swap, for example if I want window 2 to be in window 1.

```
prefix + shift + : = "swap-window -s 2(current window) -t 1 (window you want to move)  
```

close a session in tmux

```
prefix + shift + : = "detach" 
```

## Edit a file with micro command

```
cat /etc/passwd | micro
```
Create a script using micro and it will detect the syntax in bash or zsh. 

```
micro script.sh 
```
