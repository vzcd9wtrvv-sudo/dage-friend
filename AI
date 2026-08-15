export default {
 async fetch(request,env){
  const url=new URL(request.url);
  if(!url.pathname.startsWith("/api/"))return env.ASSETS.fetch(request);

  // Browser-friendly diagnostics.
  if(url.pathname==="/api/health" && request.method==="GET"){
    return json({ok:true,ai:!!env.AI,service:"dage-ai"});
  }

  if(url.pathname==="/api/ai-test" && request.method==="GET"){
    if(!env.AI)return json({ok:false,ai:false,error:"AI binding missing"},503);
    try{
      const ans=await env.AI.run("@cf/meta/llama-3.1-8b-instruct-fast",{
        prompt:"只回覆 AI_OK",
        max_tokens:16,
        temperature:0
      });
      const raw=ans?.response||ans?.result?.response||ans?.choices?.[0]?.message?.content||ans;
      return json({ok:true,ai:true,raw});
    }catch(e){
      return json({ok:false,ai:true,error:String(e?.message||e)},500);
    }
  }

  if(request.method!=="POST")return json({error:"method"},405);
  if(!env.AI)return json({error:"AI binding missing"},503);
  try{
   const body=await request.json();
   if(url.pathname==="/api/event")return json(await genEvent(env,body));
   if(url.pathname==="/api/resolve")return json(await resolveEvent(env,body));
   if(url.pathname==="/api/biography")return json(await genBiography(env,body));
   return json({error:"not found"},404);
  }catch(e){return json({error:"failed",detail:String(e?.message||e)},500)}
 }
};
function json(x,status=200){return new Response(JSON.stringify(x),{status,headers:{"content-type":"application/json;charset=UTF-8","cache-control":"no-store"}})}
async function runJSON(env,prompt){
 const ans=await env.AI.run("@cf/meta/llama-3.1-8b-instruct-fast",{
   prompt,
   max_tokens:1100,
   temperature:1.0,
   top_p:0.92,
   frequency_penalty:0.45,
   presence_penalty:0.35,
   repetition_penalty:1.08,
   seed:Math.floor(Math.random()*9999999998)+1,
   response_format:{type:"json_object"}
 });
 const raw=ans?.response||ans?.result?.response||ans?.choices?.[0]?.message?.content||ans;
 if(raw&&typeof raw==="object"&&!Array.isArray(raw))return raw;
 let s=String(raw||"").trim().replace(/```json|```/gi,"").trim();
 if(!s)throw new Error("empty AI response");
 try{return JSON.parse(s)}catch(_e){}
 const a=s.indexOf("{"),b=s.lastIndexOf("}");
 if(a>=0&&b>a){try{return JSON.parse(s.slice(a,b+1))}catch(_e){}}
 throw new Error("invalid JSON from model");
}
const PERSONA=`你是繁體中文人生模擬遊戲的即時編劇。你不是客服，也不是一般聊天機器人；你要把 NPC「大鴿」當成一個有持續人格、記憶、偏見、情緒慣性與社群關係的人。

【大鴿人格核心】
- 長期混跡棒球、遊戲、球迷與聊天社群，對群組氣氛非常敏感。
- 易怒、愛面子、嘴硬、記仇，但不是每次都暴怒；有時會裝沒事、冷處理、陰陽怪氣、突然熱情或翻舊帳。
- 很容易把模糊訊息理解成有人在影射他；怒氣與懷疑高時尤其明顯。
- 對信任的人會透露工作、感情、金錢、家庭、追星、旅行、群組恩怨等私事；信任低時則戒備、試探、反問。
- 喜歡棒球、遊戲、啦啦隊與社群話題，也可能遇到工作挫折、財務壓力、借錢、人際失敗、戀愛、結婚、分手、旅行、生病、老化、意外好運等普通人生事件。
- 他會根據「最近12季記憶」延續舊事，不得把每一季當成失憶重開。
- 禁止把「算了啦，我真的懶得講」當萬用句。除非情境真的適合，否則每次反應必須重新生成。
- 同一種事件標題、同一句台詞、同一組選項不可連續重複。

【玩家】
玩家是大鴿長期認識的「假好友」。玩家可以安慰、附和、拱火、轉移話題、拉其他人進來、冷處理、勸阻或故意讓局勢更複雜。大鴿會依信任、懷疑、怒氣和既往記憶判斷玩家動機。

【傳奇人物】
- 博士：普通傳奇人物之一。偏好棒球理論、預測、抽卡與辯論，會與大鴿互嘴，但不要每季出場。
- 伊神：高頻傳奇人物。像更衝、更失控的大鴿鏡像；短句、直接、容易把小事放大，常講「你是在講什麼」「講重點」「不爽就退」之類社群語氣。伊神出現時通常讓聊天室升溫，但也可能意外站大鴿這邊。
- 其他可穿插人物：盤咕、汪達、西雅兔哥、柳丁哥、醬財、養肌、莫提斯。
- 人物要像群組裡真的有人在講話，不要像旁白機器人。

【事件節奏】
- 每季一個主事件，事件可以很小，也可以是連續數季的大事件。
- 約 35% 普通生活事件、35% 社群/棒球/遊戲事件、20% 人際/感情/財務事件、10% 稀有重大事件。
- 大事件可跨季延續，但每季要有新進展。
- 怒氣高：提高封鎖、退群、翻舊帳、衝突、錯怪人的機率。
- 信任高：提高私密人生事件、求助、借錢、感情與家庭事件。
- 懷疑高：提高試探、截圖猜疑、群友影射、假好友曝光風險。
- 不要讓所有事情都悲慘，也不要所有事情都成功。

【敘事風格】
- 繁體中文，台灣網路社群語感。
- 自然、具體、略帶嘲諷，像荒謬但連貫的長篇人物連續劇。
- 不要寫「你的介入：」「系統判定：」這種遊戲設計語句。
- 可以描寫角色做出不成熟、尷尬或令人反感的行為及後果，但不要提供騷擾、跟蹤、偷拍、性騷擾、暴力、詐騙或其他傷害行為的操作方法。
- 只根據遊戲內虛構設定生成，不加入真實私人聯絡方式、地址或可識別個資。`;

function sanitizeDialogue(lines){
 if(!Array.isArray(lines))return [];
 return lines.slice(0,8).map((x,i)=>{
   if(typeof x==="string")return {name:i===0?"大鴿":"群友",text:x};
   if(Array.isArray(x))return {name:String(x[0]||"群友"),text:String(x[1]||"")};
   if(x&&typeof x==="object")return {name:String(x.name||x.speaker||x.role||x.character||"群友"),text:String(x.text||x.message||x.content||x.reply||"")};
   return null;
 }).filter(x=>x&&x.text);
}
function sanitizeChoices(cs){
 if(!Array.isArray(cs))return [];
 return cs.map((c,i)=>{
   if(typeof c==="string")return {id:String.fromCharCode(97+i),text:c};
   if(c&&typeof c==="object")return {id:String(c.id||String.fromCharCode(97+i)),text:String(c.text||c.label||c.choice||c.action||"")};
   return null;
 }).filter(x=>x&&x.text).slice(0,3);
}
function sanitizeEventPayload(obj){
 const ev=obj?.event&&typeof obj.event==="object"?obj.event:obj;
 if(!ev||typeof ev!=="object")throw new Error("event missing");
 const out={
  category:String(ev.category||ev.type||"人生"),
  title:String(ev.title||ev.eventTitle||ev.headline||"這一季發生了一些事"),
  description:String(ev.description||ev.story||ev.summary||ev.text||"大鴿的生活又出現新的變數。"),
  legendary:String(ev.legendary||ev.legendaryCharacter||ev.character||"無"),
  dialogue:sanitizeDialogue(ev.dialogue||ev.messages||ev.chat||[]),
  choices:sanitizeChoices(ev.choices||ev.options||ev.actions||[])
 };
 while(out.choices.length<3){
   const fill=["先安慰他","順著他的情緒","把伊神或博士扯進來"];
   const i=out.choices.length;out.choices.push({id:String.fromCharCode(97+i),text:fill[i]});
 }
 return {event:out};
}
function sanitizeResultPayload(obj){
 const r=obj?.result&&typeof obj.result==="object"?obj.result:obj;
 if(!r||typeof r!=="object")throw new Error("result missing");
 const delta={};
 for(const [k,v] of Object.entries(r.delta||{}))if(typeof v==="number"&&Number.isFinite(v))delta[k]=v;
 return {result:{
  summary:String(r.summary||r.story||r.outcome||r.description||"事情往意料之外的方向發展。"),
  reaction:String(r.reaction||r.reply||r.dageReply||r.response||"算了啦，我真的懶得講。"),
  dialogue:sanitizeDialogue(r.dialogue||r.messages||r.chat||[]),
  delta,
  flags:Array.isArray(r.flags)?r.flags.slice(0,3).map(String):[],
  unblock:r.unblock===true
 }};
}

async function genEvent(env,b){
 const s=JSON.stringify(b.state||{});
 const mode=b.mode==="blocked"?"玩家目前被大鴿封鎖，事件必須圍繞解封、共同好友傳話、等待、博士介入等，但仍只給三個選項。":"正常季度。";
 const prompt=`${PERSONA}
${mode}
目前完整遊戲狀態：
${s}
請生成下一季事件。
傳奇人物規則：
- 伊神出現率非常高，目標約 45%~65% 的季度直接出現或被提到。
- 博士只是普通傳奇人物，出現率與盤咕、汪達等相近。
- 若伊神出場，對話至少讓他說 1~3 句，而且語氣明顯比大鴿更狂暴。
如果怒氣高於75，應提高暴走、退群、封鎖、翻舊帳、群組衝突機率；如果信任高，可生成比較私人的人生事件。
務必讀取最近記憶並延續因果；同標題、同事件骨架、同台詞、同選項不得連續重複。若最近幾季都是社群衝突，優先換成工作、感情、財務、家庭、旅行或普通生活事件。
只輸出合法 JSON，不要 Markdown，不要說明文字。dialogue 每項固定使用 {"name":"人物","text":"訊息"}；choices 每項固定使用 {"id":"a","text":"選項"}。格式：
{"event":{"category":"類別","title":"事件標題","description":"80~180字自然敘述","legendary":"伊神/博士/盤咕/汪達/西雅兔哥/柳丁哥/醬財/養肌/莫提斯/無 其中一個","dialogue":[{"name":"人物","text":"訊息"}],"choices":[{"id":"a","text":"選項1"},{"id":"b","text":"選項2"},{"id":"c","text":"選項3"}]}}
dialogue 0~6 則，choices 必須正好3個。伊神若為 legendary，dialogue 必須包含伊神。`;
 return sanitizeEventPayload(await runJSON(env,prompt));
}

async function resolveEvent(env,b){
 const prompt=`${PERSONA}
現在要處理玩家剛剛做的選擇。
狀態：
${JSON.stringify(b.state||{})}
事件：
${JSON.stringify(b.event||{})}
玩家選擇：
${JSON.stringify(b.choice||{})}

請推演事情後續。程式會真的套用 delta，所以數值不要過度誇張，多數介於 -15 到 +15，極端事件可到 ±25。
大鴿 reaction 必須根據當季事件、玩家選項、怒氣、信任、懷疑與最近記憶重新寫，不可使用固定模板。怒氣高時可尖銳、嘴硬、懷疑、威脅退群或封鎖；怒氣低時可敷衍、開玩笑、裝沒事或少見地坦白。不要每次都暴怒。
如果選項牽涉博士，博士可像一般傳奇人物加入對話；如果事件或選項牽涉伊神，伊神要高機率加入對話，而且通常會讓氣氛更衝。
只輸出合法 JSON，不要 Markdown，不要說明文字。dialogue 每項固定使用 {"name":"人物","text":"訊息"}。JSON：
{"result":{"summary":"120~260字，直接描述事情後來怎麼發展，略帶嘲諷，不要寫『你的介入』","reaction":"大鴿1~4句社群式反應","dialogue":[{"name":"人物","text":"訊息"}],"delta":{"mood":0,"anger":0,"trust":0,"suspicion":0,"stress":0,"social":0,"wealth":0,"love":0,"health":0},"flags":["最多3個新記憶標籤"],"unblock":false}}
不相關的 delta 欄位可省略。`;
 return sanitizeResultPayload(await runJSON(env,prompt));
}

async function genBiography(env,b){
 const prompt=`${PERSONA}
遊戲已結束。請根據以下狀態與人生記憶，替這一世的大鴿生成結局。
狀態：
${JSON.stringify(b.state||{})}
重要記憶：
${JSON.stringify(b.memories||[])}
強制結局：
${b.forcedEnding||"無"}

輸出 JSON：
{"title":"具特色的結局稱號","rank":"SSS/SS/S/A/B/C/D/F 其中一個","biography":"700~1400字繁體中文傳記。要真的整理大鴿的人生，不是只評玩家；分青年、中年、晚年或重要階段敘述，穿插伊神、博士與其他傳奇人物，以及工作、感情、金錢、棒球、封鎖與重大轉折。若伊神在此局頻繁出現，要讓他成為傳記中的重要亂源或宿敵型配角；博士則只是一般傳奇人物之一。語氣像荒謬人物傳記，略帶嘲諷但不要只是羞辱。最後再用一小段揭露玩家究竟扮演了什麼角色。"}`
 return await runJSON(env,prompt);
}
