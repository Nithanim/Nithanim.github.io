---
layout: posts
title:  "8th Wonder of the World: Assistant enhancment"
date:   1970-01-01 03:00:00 +0100
author: Nithanim
categories: games
---

The following modifications are Cheat Engine scripts and only work with  8th Wonder Of The World/Weltwunder version 1.00 Nov 29 2003.
One increases the limit of boys one can request which is normally limited to 20.
The other script fixes the oversight that the ctrl key can be used to increase the number 10 at a time instead of one. For example this is possible in carts with resources but for some reason not in the assistant.

## Higher limit for boys

```
[ENABLE]
//code from here to '[DISABLE]' will be used to enable
alloc(newmem,25)
label(returnhere)
label(originalcode)
label(exit)

newmem:
//esi: limit
//ecx: user requested boy quantity
mov esi,#100 //there is another check somewhere which seems to forbid higher values; it is not 100 but some higher value I can't remember

originalcode:
cmp ecx,esi
jnge Game.exe+18C4A
mov ecx,esi

exit:
jmp returnhere

"Game.exe"+18C44:
jmp newmem
nop
returnhere:

 
[DISABLE]
//code from here till the end of the code will be used to disable
dealloc(newmem)
"Game.exe"+18C44:
cmp ecx,esi
jnge Game.exe+18C4A
mov ecx,esi
//Alt: db 3B CE 7C 02 8B CE
```

## Ctrl for boys

```
[ENABLE]
alloc(newmem,100)
alloc(storage,1)
label(returnhere)
label(notpressed)

newmem:
//ecx: new value
//edx: old value

mov ecx,[ebp+10] //original code

mov [storage],ecx


pushad
pushfd
push 11 //ctrl
call GetAsyncKeyState
shr ax,#15
cmp ax,1
jne notpressed


push ebx
mov ebx,[storage]
pushfd
imul ebx,ebx,#10
popfd
mov [storage],ebx
pop ebx


notpressed:
popfd
popad

mov ecx,[storage]
add ecx,edx //original code


jmp returnhere


//at original place
"Game.exe"+18C39:
jmp newmem
returnhere:


[DISABLE]
dealloc(newmem)
dealloc(storage)
"Game.exe"+18C39:
mov ecx,[ebp+10]
add ecx,edx
//Alt: db 8B 4D 10 01 D1
```