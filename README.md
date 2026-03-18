# fediwallFachtagung
fediwallFachtagung
<!doctype html>
<html lang="de">
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>Fachtagung „Lernort Schule im digitalen Wandel“ – Fediwall</title>
<style>
body{margin:0;font-family:system-ui,sans-serif;background:#0f1220;color:white}
header{padding:18px;background:#1a1f3a;display:flex;justify-content:space-between;align-items:center}
h1{font-size:1.4rem;margin:0}
.wall{display:grid;grid-template-columns:repeat(auto-fill,minmax(320px,1fr));gap:18px;padding:20px}
.card{background:#1c2145;border-radius:16px;padding:16px}
.user{display:flex;align-items:center;margin-bottom:10px}
.avatar{width:42px;height:42px;border-radius:50%;margin-right:10px}
.name{font-weight:bold}
.text{line-height:1.4;margin-bottom:10px}
.media img{width:100%;border-radius:10px;margin-top:8px}
.meta{font-size:0.8rem;opacity:.7}
button{margin-left:8px;padding:8px 14px;border-radius:20px;border:none;cursor:pointer}
</style>
</head>
<body>

<header>
<h1>Fachtagung „Lernort Schule im digitalen Wandel“ – Fediwall</h1>
<div>
<button onclick="loadPosts()">Aktualisieren</button>
<button onclick="toggleFullscreen()">Vollbild</button>
</div>
</header>

<div class="wall" id="wall"></div>

<script>

const CONFIG = {

hashtags:[
"LernortSchule",
"DigitaleSchule",
"Fachtagung",
"FachtagungRostock",
"ZukunftBeruflicheBildung",
"KompetenzZentrumBeruflicheSchulen"
],

instances:[
"mastodon.social",
"chaos.social",
"troet.cafe",
"mastodon.schule"
],

refreshMs:30000,
maxPosts:30,
recentHours:48
}

let posts=[]

function strip(html){
let d=new DOMParser().parseFromString(html,"text/html")
return d.body.textContent
}

function recent(date){
let limit=Date.now()-CONFIG.recentHours*3600000
return new Date(date).getTime()>limit
}

async function fetchTag(instance,tag){
let url=`https://${instance}/api/v1/timelines/tag/${tag}?limit=20`
let r=await fetch(url)
return await r.json()
}

async function loadPosts(){

let map=new Map()

for(let instance of CONFIG.instances){

for(let tag of CONFIG.hashtags){

try{

let items=await fetchTag(instance,tag)

for(let s of items){

let post=s.reblog||s
if(!recent(post.created_at))continue

let id=post.url
if(map.has(id))continue

map.set(id,{

user:post.account.display_name||post.account.username,
acct:post.account.acct,
avatar:post.account.avatar,

text:strip(post.content),

media:post.media_attachments
.filter(m=>m.type=="image")
.map(m=>m.preview_url),

url:post.url,
date:post.created_at

})

}

}catch(e){
console.log(instance,tag,e)
}

}

}

posts=[...map.values()]
.sort((a,b)=>new Date(b.date)-new Date(a.date))
.slice(0,CONFIG.maxPosts)

render()

}

function render(){

let wall=document.getElementById("wall")
wall.innerHTML=""

for(let p of posts){

let mediaHTML=p.media.map(m=>`<img src="${m}">`).join("")

wall.innerHTML+=`
<div class="card">

<div class="user">
<img class="avatar" src="${p.avatar}">
<div>
<div class="name">${p.user}</div>
<div class="meta">@${p.acct}</div>
</div>
</div>

<div class="text">${p.text}</div>

<div class="media">${mediaHTML}</div>

<div class="meta">
<a href="${p.url}" target="_blank">Beitrag öffnen</a>
</div>

</div>
`
}

}

setInterval(loadPosts,CONFIG.refreshMs)

function toggleFullscreen(){
if(!document.fullscreenElement)
document.documentElement.requestFullscreen()
else
document.exitFullscreen()
}

loadPosts()

</script>

</body>
</html>
