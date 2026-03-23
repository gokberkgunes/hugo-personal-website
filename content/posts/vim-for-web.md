---
title: "Vim for Web"
date: 2026-03-23T01:35:39-05:00
authors: ["Gokberk Gunes", ]
tags: []
header_image:
draft: false
---

## Tridactyl
Tridactyl is closest thing to a terminal + vim + emacs experience in Firefox.
Imagine, you can use `Ctrl + W` to delete words! Even though it doesn't have
a builtin search that can be used with `/` followed by `n` or `N`, this is
better than other alternatives.

### Scrolling
Sometimes, `gg` and `G` doesn't work properly. To fix this, I use below,
```js
bind gg js window._triScrl && (window._triScrl === window || window._triScrl.isConnected) ? window._triScrl.scrollTo(0,0) : ((t,m)=>(document.querySelectorAll('*').forEach(e=>((window.getComputedStyle(e).overflowY||"").match(/auto|scroll|overlay/)&&e.scrollHeight>e.clientHeight&&e.clientWidth*e.clientHeight>m)&&(m=e.clientWidth*e.clientHeight,t=e)), window._triScrl = t || window, window._triScrl.scrollTo(0,0)))(null,window.innerWidth*window.innerHeight*0.4)

bind G  js window._triScrl && (window._triScrl === window || window._triScrl.isConnected) ? window._triScrl.scrollTo(0,window._triScrl.scrollHeight||document.documentElement.scrollHeight) : ((t,m)=>(document.querySelectorAll('*').forEach(e=>((window.getComputedStyle(e).overflowY||"").match(/auto|scroll|overlay/)&&e.scrollHeight>e.clientHeight&&e.clientWidth*e.clientHeight>m)&&(m=e.clientWidth*e.clientHeight,t=e)), window._triScrl = t || window, window._triScrl.scrollTo(0,window._triScrl.scrollHeight||document.documentElement.scrollHeight)))(null,window.innerWidth*window.innerHeight*0.4)
```

I rebind `j` and `k`  to below, which work better than default when it comes to
understand focused elements.

```js
bind j  js window._triScrl && (window._triScrl === window || window._triScrl.isConnected) ? window._triScrl.scrollBy(0,200) : ((t,m)=>(document.querySelectorAll('*').forEach(e=>((window.getComputedStyle(e).overflowY||"").match(/auto|scroll|overlay/)&&e.scrollHeight>e.clientHeight&&e.clientWidth*e.clientHeight>m)&&(m=e.clientWidth*e.clientHeight,t=e)), window._triScrl = t || window, window._triScrl.scrollBy(0,200)))(null,window.innerWidth*window.innerHeight*0.4)

bind k  js window._triScrl && (window._triScrl === window || window._triScrl.isConnected) ? window._triScrl.scrollBy(0,-200) : ((t,m)=>(document.querySelectorAll('*').forEach(e=>((window.getComputedStyle(e).overflowY||"").match(/auto|scroll|overlay/)&&e.scrollHeight>e.clientHeight&&e.clientWidth*e.clientHeight>m)&&(m=e.clientWidth*e.clientHeight,t=e)), window._triScrl = t || window, window._triScrl.scrollBy(0,-200)))(null,window.innerWidth*window.innerHeight*0.4)
```


I rebind `C-d` and `C-u` to `d` and `u` due to laziness.

```js
bind d  js window._triScrl && (window._triScrl === window || window._triScrl.isConnected) ? window._triScrl.scrollBy(0,  window._triScrl.clientHeight/2||window.innerHeight/2)  : ((t,m)=>(document.querySelectorAll('*').forEach(e=>((window.getComputedStyle(e).overflowY||"").match(/auto|scroll|overlay/)&&e.scrollHeight>e.clientHeight&&e.clientWidth*e.clientHeight>m)&&(m=e.clientWidth*e.clientHeight,t=e)), window._triScrl = t || window, window._triScrl.scrollBy(0,window._triScrl.clientHeight/2||window.innerHeight/2)))(null,window.innerWidth*window.innerHeight*0.4)

bind u  js window._triScrl && (window._triScrl === window || window._triScrl.isConnected) ? window._triScrl.scrollBy(0,-(window._triScrl.clientHeight/2||window.innerHeight/2)) : ((t,m)=>(document.querySelectorAll('*').forEach(e=>((window.getComputedStyle(e).overflowY||"").match(/auto|scroll|overlay/)&&e.scrollHeight>e.clientHeight&&e.clientWidth*e.clientHeight>m)&&(m=e.clientWidth*e.clientHeight,t=e)), window._triScrl = t || window, window._triScrl.scrollBy(0,-(window._triScrl.clientHeight/2||window.innerHeight/2))))(null,window.innerWidth*window.innerHeight*0.4)


bind x tabclose

bind X undo
```



### Hints
Unlike `Vimium` and `SurfingKeys`, `Tridactyl` doesn't focus on topmost
window. In other words, it doesn't care about CSS z-scores. Moreover it may
show uneccessary hints when pressed `f`.  For now, I use below
to mimic `Vimium` like behavior. this though this version rarely fails to reach some elements. Thereforee,
I rebind defaulf `f` and `F` to `gf` and `gF`. Furthermore, even when these can't
reach, I use `;;`.
```js
bind f js (()=>{let S='a,button,input,textarea,select,summary,iframe,[role="button"],[role="menuitem"],[role="link"],[tabindex],.cursor-pointer,[href],[onclick],faceplate-tracker,shreddit-async-loader';document.querySelectorAll('.tri-proxy').forEach(e=>e.remove());document.querySelectorAll('.tri-vis').forEach(e=>e.classList.remove('tri-vis'));let m=[];let Q=r=>{let a=Array.from(r.querySelectorAll('*'));return a.reduce((c,e)=>(e.shadowRoot?c.concat(Q(e.shadowRoot)):c),a)};let C=(p,c)=>{let u=c;while(u){if(u===p)return true;u=u.assignedSlot||u.parentNode||u.host}return false};Q(document).filter(e=>e.matches&&e.matches(S)).reverse().forEach(e=>{let t=e.getBoundingClientRect(),s=window.getComputedStyle(e);if(t.width>0&&t.height>0&&t.right>0&&t.bottom>0&&t.left<window.innerWidth&&t.top<window.innerHeight&&s.visibility!=='hidden'&&s.opacity!=='0'&&s.pointerEvents!=='none'){if(m.some(r=>Math.abs(r.x-t.left)<5&&Math.abs(r.y-t.top)<5&&Math.abs(r.w-t.width)<5&&Math.abs(r.h-t.height)<5))return;let h=[[t.width/2,t.height/2],[5,5],[t.width-5,t.height-5]].some(([x,y])=>{let l=document.elementFromPoint(t.left+x,t.top+y);if(!l)return false;while(l.shadowRoot){let n=l.shadowRoot.elementFromPoint(t.left+x,t.top+y);if(!n||n===l)break;l=n}return C(e,l)||C(l,e)});if(h){m.push({x:t.left,y:t.top,w:t.width,h:t.height});if(e.getRootNode()!==document){let p=document.createElement('div');p.className='tri-vis tri-proxy';p.style.cssText=`position:fixed;left:${t.left}px;top:${t.top}px;width:${t.width}px;height:${t.height}px;z-index:999999;cursor:pointer;background:transparent;`;p.onclick=ev=>{ev.preventDefault();e.click()};document.documentElement.appendChild(p)}else{e.classList.add('tri-vis')}}}});tri.excmds.hint('-Jc','.tri-vis:not(svg,img,path)')})()

bind F js (()=>{let S='a,button,input,textarea,select,summary,iframe,[role="button"],[role="menuitem"],[role="link"],[tabindex],.cursor-pointer,[href],[onclick],faceplate-tracker,shreddit-async-loader';document.querySelectorAll('.tri-proxy').forEach(e=>e.remove());document.querySelectorAll('.tri-vis').forEach(e=>e.classList.remove('tri-vis'));let m=[];let Q=r=>{let a=Array.from(r.querySelectorAll('*'));return a.reduce((c,e)=>(e.shadowRoot?c.concat(Q(e.shadowRoot)):c),a)};let C=(p,c)=>{let u=c;while(u){if(u===p)return true;u=u.assignedSlot||u.parentNode||u.host}return false};Q(document).filter(e=>e.matches&&e.matches(S)).reverse().forEach(e=>{let t=e.getBoundingClientRect(),s=window.getComputedStyle(e);if(t.width>0&&t.height>0&&t.right>0&&t.bottom>0&&t.left<window.innerWidth&&t.top<window.innerHeight&&s.visibility!=='hidden'&&s.opacity!=='0'&&s.pointerEvents!=='none'){if(m.some(r=>Math.abs(r.x-t.left)<5&&Math.abs(r.y-t.top)<5&&Math.abs(r.w-t.width)<5&&Math.abs(r.h-t.height)<5))return;let h=[[t.width/2,t.height/2],[5,5],[t.width-5,t.height-5]].some(([x,y])=>{let l=document.elementFromPoint(t.left+x,t.top+y);if(!l)return false;while(l.shadowRoot){let n=l.shadowRoot.elementFromPoint(t.left+x,t.top+y);if(!n||n===l)break;l=n}return C(e,l)||C(l,e)});if(h){m.push({x:t.left,y:t.top,w:t.width,h:t.height});if(e.getRootNode()!==document){let p=document.createElement('div');p.className='tri-vis tri-proxy';p.style.cssText=`position:fixed;left:${t.left}px;top:${t.top}px;width:${t.width}px;height:${t.height}px;z-index:999999;cursor:pointer;background:transparent;`;p.onclick=ev=>{ev.preventDefault();e.click()};document.documentElement.appendChild(p)}else{e.classList.add('tri-vis')}}}});tri.excmds.hint('-bJc','.tri-vis:not(svg,img,path)')})()
```

### Focus on Text Boxes
There's `gi` command for last focused text inputs; but we can have a selector
for text inputs:

```js
bind i hint -c input:not([type=hidden]):not([type=radio]):not([type=checkbox]):not([type=submit]):not([type=button]), textarea, [contenteditable=true], [role=textbox]

bind I hint -Jc [contenteditable],input,textarea,select
```
Note: If there's a single text input it'll focus directly.


### Delete Like God Intended
#### Words
Open `about:keyboard` then delete bind `Ctrl + W` to `Alt + W`.
```js
bind --mode=ex <C-w> text.backward_kill_word

bind --mode=insert <C-w> text.backward_kill_word
```

#### Lines

```js
bind --mode=ex <C-u> text.backward_kill_line

bind --mode=insert <C-u> text.backward_kill_line
```


## Surfingkeys
It does poorly with `find` mode well on `Firefox`. When you press `n` and `N`,
you enter `visual` or `caret` mode. This means if you want to scroll up and down,
now you scroll up and down by a line.

Second, it doesn't center the viewport and the settings to fix this doesn't work.
(https://github.com/brookhong/Surfingkeys/issues/2034)

Third, you need to know javascript to edit configuration file.

## Vimium or Vimium C
Best of out of the box, not as customizable as `Tridactyl` and `SurfingKeys`.
Hints of `Vimium C`  work better on javascript based elements like Cloudflate's
CAPTCHA compared to `Vimium`.
