<!DOCTYPE html>
<html lang="en"><head><meta charset="utf-8">
<meta name="viewport" content="width=device-width,initial-scale=1,viewport-fit=cover">
<title>JARVIS</title>
<style>
:root{--c:#00e5ff;--bg:#02070d}
*{box-sizing:border-box}
html,body{height:100%;margin:0;background:radial-gradient(circle at 50% 25%,#07283a,var(--bg) 70%);color:var(--c);font-family:-apple-system,Helvetica,sans-serif}
body{display:flex;flex-direction:column;padding:env(safe-area-inset-top) 12px env(safe-area-inset-bottom)}
header{display:flex;justify-content:space-between;align-items:center;padding:8px 0;letter-spacing:5px;font-weight:700}
button{background:none;border:1px solid var(--c);color:var(--c);border-radius:10px;padding:8px 12px;font-size:15px}
#core{flex:none;width:130px;height:130px;margin:4px auto 10px;border-radius:50%;border:3px dashed var(--c);box-shadow:0 0 25px var(--c),inset 0 0 25px var(--c);display:flex;align-items:center;justify-content:center;animation:spin 10s linear infinite}
#core i{width:54px;height:54px;border-radius:50%;background:radial-gradient(#fff,var(--c));box-shadow:0 0 30px var(--c);animation:glow 2.5s ease-in-out infinite}
#core.listen{border-color:#ff3d57;box-shadow:0 0 35px #ff3d57,inset 0 0 25px #ff3d57;animation-duration:2s}
#core.talk i{animation-duration:.5s}
@keyframes spin{to{transform:rotate(360deg)}}
@keyframes glow{50%{transform:scale(1.25);opacity:.7}}
#log{flex:1;overflow-y:auto;padding:6px 2px;font-size:16px;line-height:1.4}
#log div{margin:6px 0;padding:8px 12px;border:1px solid #0a4a5a;border-radius:12px;background:#031a24;max-width:88%}
.me{margin-left:auto!important;border-color:var(--c)!important;color:#fff}
.sys{color:#ff9d00;border-color:#6b4300!important}
#bar{display:flex;gap:8px;padding:8px 0}
#txt{flex:1;min-width:0;background:#031a24;border:1px solid var(--c);color:#fff;border-radius:10px;padding:10px;font-size:16px}
#mic{width:56px;border-radius:50%;font-size:22px;box-shadow:0 0 12px var(--c)}
</style></head><body>
<header><span>J.A.R.V.I.S</span><span><button id="lg">EN</button> <button id="st">⚙</button></span></header>
<div id="core"><i></i></div>
<div id="log"></div>
<div id="bar"><input id="txt" placeholder="Type or tap the mic…"><button id="send">➤</button><button id="mic">🎙</button></div>
<script>
const MODELS=["gemini-2.5-flash","gemini-2.5-flash-lite","gemini-2.0-flash"];
const $=id=>document.getElementById(id);
let lang=localStorage.lang||"en",key=localStorage.key||"",gkey=localStorage.gkey||"",history=[],notes=JSON.parse(localStorage.notes||"[]"),contacts=JSON.parse(localStorage.contacts||"{}");
const SC={alarm:"Jarvis Alarm",remind:"Jarvis Remind",torch_on:"Jarvis Torch On",torch_off:"Jarvis Torch Off",wifi_on:"Jarvis WiFi On",wifi_off:"Jarvis WiFi Off"};
const LB={call:"Call",sms:"Open SMS",wa:"Open WhatsApp",mail:"Open Mail",alarm:"Set alarm",remind:"Add reminder",torch_on:"Torch ON",torch_off:"Torch OFF",wifi_on:"Wi-Fi ON",wifi_off:"Wi-Fi OFF"};
const SR=window.SpeechRecognition||window.webkitSpeechRecognition;
$("lg").textContent=lang==="ur"?"اردو":"EN";

function add(cls,t){const d=document.createElement("div");d.className=cls;d.textContent=t;$("log").appendChild(d);$("log").scrollTop=1e9}

function speak(t){
  speechSynthesis.cancel();
  const ur=/[\u0600-\u06FF]/.test(t),u=new SpeechSynthesisUtterance(t);
  u.lang=ur?"ur-PK":"en-GB";
  const F=/samantha|serena|ava|allison|susan|karen|moira|tessa|kate|stephanie|fiona|zoe|nicky|uzma|female/i;
  const vs=speechSynthesis.getVoices().filter(v=>v.lang.replace("_","-").toLowerCase().startsWith(ur?"ur":"en"));
  const v=vs.find(x=>F.test(x.name))||vs[0];
  if(v)u.voice=v;
  u.pitch=1.05;u.rate=.95;
  $("core").className="talk";u.onend=()=>$("core").className="";
  speechSynthesis.speak(u);
}

function local(q){
  const s=q.toLowerCase();let m;
  m=q.match(/^contact\s+(.+?)\s+(\S+@\S+|\+?\d[\d\s-]{6,})$/i);
  if(m){contacts[m[1].toLowerCase()]=m[2].replace(/\s|-/g,"");localStorage.contacts=JSON.stringify(contacts);return "Saved "+m[1]+", sir.";}
  if(/\btime\b|وقت|ٹائم/.test(s)&&!/timer|alarm/.test(s))return "It is "+new Date().toLocaleTimeString([],{hour:"numeric",minute:"2-digit"})+", sir.";
  if(/\bdate\b|\btoday\b|تاریخ|آج/.test(s)&&s.length<30)return "Today is "+new Date().toLocaleDateString([],{weekday:"long",day:"numeric",month:"long",year:"numeric"})+".";
  m=s.match(/(\d+)\s*(minute|min|second|sec|منٹ|سیکنڈ)/);
  if(m&&/timer|ٹائمر/.test(s)){
    const n=+m[1],ms=n*(/^(m|منٹ)/.test(m[2])?60000:1000);
    setTimeout(()=>{add("ai","⏰ Timer finished, sir.");speak("Your timer is finished, sir.")},ms);
    return "Timer set for "+n+" "+m[2]+", sir.";
  }
  m=q.match(/^(?:note|remember)[:\s]+(.+)/i);
  if(m){notes.push(m[1]);localStorage.notes=JSON.stringify(notes);return "Noted, sir.";}
  if(/show.*notes|my notes/.test(s))return notes.length?notes.map((n,i)=>(i+1)+". "+n).join(" "):"You have no notes, sir.";
  return null;
}

function sys(){
  return "You are JARVIS, a witty, polite personal AI assistant like Tony Stark's. Reply ONLY with JSON: {\"say\":\"...\",\"act\":{\"t\":\"none\",\"to\":\"\",\"text\":\"\",\"subject\":\"\"}}. say = 1-3 short spoken sentences, no markdown or emojis, in the user's language (Urdu in Urdu script, even if the user wrote Roman Urdu). Address the user as sir (Urdu: janab). act.t is none unless the user asks for an action; then use one of: call, sms, wa, mail, alarm, remind, torch_on, torch_off, wifi_on, wifi_off. to = contact name, phone number or email. text = the full message or email body that you write yourself in the user's language, or the reminder text, or the alarm time as HH:MM 24-hour. subject = email subject. If the user dictates an email and asks for a reply, write the reply in act.text with t mail. Known contacts: "+(Object.keys(contacts).join(", ")||"none")+". "+(lang==="ur"?"The user is speaking Urdu. ":"")+"Current date and time: "+new Date().toString();
}

function parse(t){
  try{const o=JSON.parse(t.replace(/```json|```/g,"").trim());return {say:o.say||"Done, sir.",act:o.act||{t:"none"}}}
  catch(e){return {say:t,act:{t:"none"}}}
}

async function call(p){
  let r,j;
  if(p==="groq"){
    const msgs=[{role:"system",content:sys()}].concat(history.slice(-12).map(h=>({role:h.role==="model"?"assistant":"user",content:h.parts[0].text})));
    r=await fetch("https://api.groq.com/openai/v1/chat/completions",{method:"POST",headers:{"Content-Type":"application/json","Authorization":"Bearer "+gkey},body:JSON.stringify({model:"llama-3.3-70b-versatile",messages:msgs,max_tokens:500,response_format:{type:"json_object"}})});
    j=await r.json();
    if(!r.ok)throw new Error(j.error?.message||r.status);
    return j.choices?.[0]?.message?.content||"";
  }
  r=await fetch("https://generativelanguage.googleapis.com/v1beta/models/"+p+":generateContent?key="+key,{method:"POST",headers:{"Content-Type":"application/json"},body:JSON.stringify({systemInstruction:{parts:[{text:sys()}]},contents:history.slice(-12),generationConfig:{responseMimeType:"application/json"}})});
  j=await r.json();
  if(!r.ok)throw new Error(j.error?.message||r.status);
  return (j.candidates?.[0]?.content?.parts||[]).map(x=>x.text).join("");
}

async function ask(q){
  const tries=(key?MODELS:[]).concat(gkey?["groq"]:[]);
  if(!tries.length)return {say:"Please tap the gear icon and paste your free API key first, sir."};
  history.push({role:"user",parts:[{text:q}]});
  let last="";
  for(let round=0;round<2;round++){
    for(const p of tries){
      try{
        const o=parse(await call(p));
        history.push({role:"model",parts:[{text:o.say}]});
        return o;
      }catch(e){last=e.message||"network problem";}
    }
    await new Promise(x=>setTimeout(x,1500));
  }
  history.pop();
  return {say:"Sorry sir, the AI servers are busy. Please try again in a minute. ("+last+")"};
}

function who(x){x=(x||"").trim();return contacts[x.toLowerCase()]||x}

function url(a){
  const e=encodeURIComponent,t=a.text||"",to=who(a.to),d=to.replace(/[^\d+]/g,"");
  if(a.t==="call")return d?"tel:"+d:"";
  if(a.t==="sms")return d?"sms:"+d+"&body="+e(t):"";
  if(a.t==="wa")return d?"https://wa.me/"+d.replace(/\D/g,"")+"?text="+e(t):"";
  if(a.t==="mail")return "mailto:"+(to.includes("@")?to:"")+"?subject="+e(a.subject||"")+"&body="+e(t);
  if(SC[a.t])return "shortcuts://run-shortcut?name="+e(SC[a.t])+"&input=text&text="+e(t);
  return "";
}

function show(o){
  const a=o.act;if(!a||!a.t||a.t==="none")return;
  const u=url(a);
  if(!u){add("sys","No number saved for \""+(a.to||"?")+"\". Type: contact Name +923001234567");return}
  if(a.text&&["sms","wa","mail"].includes(a.t))add("sys","✎ "+a.text);
  const b=document.createElement("button");
  b.textContent="▶ "+(LB[a.t]||a.t)+(a.to?" → "+a.to:"");
  b.style.margin="4px 0";b.onclick=()=>{location.href=u};
  $("log").appendChild(b);$("log").scrollTop=1e9;
}

async function handle(q){
  q=q.trim();if(!q)return;
  add("me",q);
  const l=local(q),o=l?{say:l}:await ask(q);
  add("ai",o.say);speak(o.say);show(o);
}

function listen(){
  if(!SR){add("sys","Voice input is not available here. Use the 🎙 key on your iPhone keyboard to dictate instead.");return}
  speechSynthesis.speak(new SpeechSynthesisUtterance(""));
  const r=new SR();
  r.lang=lang==="ur"?"ur-PK":"en-US";
  r.onstart=()=>$("core").className="listen";
  r.onend=()=>{if($("core").className==="listen")$("core").className=""};
  r.onresult=e=>handle(e.results[0][0].transcript);
  r.onerror=e=>add("sys","Mic error: "+e.error);
  r.start();
}

$("mic").onclick=listen;
$("send").onclick=()=>{speechSynthesis.speak(new SpeechSynthesisUtterance(""));const v=$("txt").value;$("txt").value="";handle(v)};
$("txt").onkeydown=e=>{if(e.key==="Enter")$("send").click()};
$("lg").onclick=()=>{lang=lang==="en"?"ur":"en";localStorage.lang=lang;$("lg").textContent=lang==="ur"?"اردو":"EN"};
$("st").onclick=()=>{
  const k=prompt("Gemini API key (optional):",key);
  if(k!==null){key=k.trim();localStorage.key=key}
  const g=prompt("Groq API key (backup, recommended):",gkey);
  if(g!==null){gkey=g.trim();localStorage.gkey=gkey}
};
add("ai","Good day, sir. JARVIS online. I can call, message, set alarms and more. Tap the mic.");
</script></body></html>
