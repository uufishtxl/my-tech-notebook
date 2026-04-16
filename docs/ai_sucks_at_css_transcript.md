# 数据流向

- Active Production Forge（主动锻造炉）：肌肉记忆系统。里面是需要能够“脱口而出”的，目标是 Fluency。
- Contextual Library / Wordbook （语境索引库）：单词本，需要做到“见词知意”，目标是理解 Comprehension。

比如：

| **词汇**         | **建议去向**               | **判定理由 (Auditor's Verdict)**                                             |
| -------------- | ---------------------- | ------------------------------------------------------------------------ |
| **Suck at...** | **Production Forge**   | **高频替换词**。它是你告别“中式英语/教科书英语”的捷径。用它替换脑子里的 "I am not good at"，瞬间提升表达的“原生感”。 |
| **Potluck**    | **Contextual Library** | **文化背景词**。除非你打算在美国组织聚餐或主持播客，否则它的主动使用率极低。理解它的“随机、凑份子”含义即可。                |

### Production Forge
播客中的句子感觉最好不要直接引用，可能会比较破碎，可以让LLM返回一个根据原文清理后的句子。

```JSON
{
    "label": "suck at",
    "acquisition_path": "forge",
    "grain_size": "phrase",
    "tag": "cement",
    "replacement_trigger": "not good at / be poor at",
    "explanation": "极其不擅长某事，或某物性能极差。在工程师文化中是表达“水平有限”或“吐槽工具”的首选，比 'not good at' 更具原生感（Native-like）。",
    "etymology_hint": "俚语起源，原指“吸走空气/能量”，引申为由于表现太差而让人感到乏味或失望。无需深究词源，重在语感。",
    "example": "I'm okay with Python, but I absolutely suck at centering things with CSS.",
    "scenarios": [
        {
            "description": "Dev Stand-up (自嘲)",
            "content": "I still suck at remembering these complex Docker CLI commands."
        },
        {
            "description": "Tool Evaluation (吐槽)",
            "content": "This legacy library sucks at handling concurrent requests."
        }
    ],
    "mastery": 0,
    "box_level": 1,
    "next_review_at": "2026-04-10T14:00:00Z",
    "status": "SUCCESS"
}
```

```JSON
{
    "label": "potluck",
    "acquisition_path": "library",
    "grain_size": "word",
    "tag": "vibe",
    "replacement_trigger": "", 
    "explanation": "原指“百家餐”聚会（每个人带一个菜分享）。在技术播客或社区中，常指“随机提问/杂谈环节”，强调内容的随机性和社区驱动性。",
    "etymology_hint": "【锅里的运气】：Pot (锅) + Luck (运气)。起源于招待不速之客，锅里有什么就吃什么，全凭运气。记住了“锅里盲盒”的感觉，就记住了这个词。",
    "example": "Welcome to another potluck episode, where we answer the questions you sent in!",
    "scenarios": [
        {
            "description": "Podcast/Community (社区互动)",
            "content": "Today's show is a potluck format—no set agenda, just your questions."
        },
        {
            "description": "Social Event (职场社交)",
            "content": "The team is having a potluck lunch this Friday; feel free to bring your favorite dish."
        }
    ],
    "mastery": 0,
    "box_level": 1,
    "next_review_at": "2026-05-10T14:00:00Z",
    "status": "SUCCESS"
}
```
`scenarios`这个字段还需要保留吗？
### Wordbook
- 包含“记忆技巧“，比如 potluck 一词：
	- 起源：最初是指给“不速之客”提供的事务。家里锅里正在煮什么，客人就吃什么。吃到好肉还是烂菜，全凭运气（luck of the pot）
	- 演变：后来变成一种社交聚会方式——每个人带一道菜放在“锅”里
	- 技术引申：指 Potluck Questions，即听众随便提问，主持人像开盲盒一样随机问答。
	- 场景联想：A potluck episode 表示播客节目中抽问题回答；或者 LLM 中也可以使用：Vibe coding with different LLMs is like a **potluck**; you throw a bunch of loose prompts into the mix, and you’re never quite sure if the final product will be a masterpiece or a mess.

# Podcast Transcript




- **[00:00]** Welcome to syntax today we got a <span style="color: red">potluck</span> episodes as where you bring the questions we bring the answers got some really good questions today why does AI <span style="color: red">suck</span> so much <span style="color: red">at</span> CSS how are we supposed to get good clean repeatable design out of using LLM some questions around vibe rules skills versus rules versus agents should
- **[00:19]** these rules be shipped with a package or should you just reach out to the internet to grab them interviewing with AI a genius genius use of using a dumb model from Brendan Falk.
- a potluck: 这是完全没接触过的词语
- suck at...
- get...out of...：能看懂，但是自己不会对简单的词来灵活使用

| **词汇/短语**             | **标签**        | **深度解析**                                                                                      |
| --------------------- | ------------- | --------------------------------------------------------------------------------------------- |
| **`Opinionated`**     | **Industry**  | **技术圈最高频词之一**。指工具“有主见/强约束”。比如 Django 是 opinionated，因为它规定了目录结构；而 Flask 则不。这是 Tech Writer 的必修词。 |
| **`Table stakes`**    | **Industry**  | 原意是赌博的入场筹码。在技术里指“**最基本的门槛**”。“AI 能写出代码只是 table stakes（基操），能写好才是本事。”                           |
| **`A rigid system`**  | **Industry**  | **Rigid (僵硬/刻板)**。形容系统非常死板、不灵活。跟它对应的词通常是 `Flexible`。                                          |
| **`One-off classes`** | **Industry**  | **One-off (一次性的)**。在 CSS 里指为了调一个间距专门写一个类，以后再也不用了。这是“脏代码”的典型表现。                                |
| **`Hit sb with...`**  | **Casual**    | **非常地道**。相当于“把...扔给某人”。"Hit us with the first question"（把第一个问题抛给我们吧）。                         |
| **`Kick out`**        | **Explainer** | **水泥动词**。指“产出/输出”。“The LLM kicked out a lot of garbage.”（LLM 吐了一堆垃圾）。                         |
- **[00:32]** We're going to describe his thoughts on that which I thought it was so so genius.
- **[00:37]** How did you find and debug performance issues in your application?
- **[00:41]** There's so many different spots where your app can start to feel slow.
- **[00:45]** from someone says, why isn't stuff we're getting better and faster with AI?
- **[00:50]** Isn't that what we were promised?
- **[00:52]** Let's get into it.
- **[00:53]** What's up, Scott?
- **[00:54]** You want to hit us with the first question?
- - hit sb with sth
- **[00:55]** Yeah, first question from Yergen.
- **[00:57]** Do you have any workflows suggestions for design and CSS when working with AI agentic coding?
- - AI agentic coding:  能看懂，但是我可能只会说 ai coding，不太习惯这种组合方式
- **[01:05]** I'm currently experimenting with open spec plus cloud code and targeted system prompts for roles and it's great at functionality, but the design looks terrible.
- **[01:17]** I have started a new experiment with new system prompts for UX design and opinionated front-end dev persona, but I'm wondering how other people are tackling this.
- **[01:28]** Is CSS without a rigid design system just too difficult for LLMs?
- **[01:34]** I will say this, Yergen, even with a rigid design system.
- cloud code 应该是 claude code，这里是 whisper 无法抓住的“新词”，这可以无视，也可以通过目前我用 langgraph 建立的自然语言修改 SQL 数据来手动修改。
- open spec 是一个新东西吗？不太清楚是什么。阅读的时候可能也会经常碰到这种情况，所以划到的不一定是新词，可能只是一个新的概念。而概念就意味着不需要进入肌肉记忆学习系统
- opinionated：这属于词根很熟悉，但是对词语的意思完全没有概念的天堑一样的存在，经常会有类似的词语。
- persona 这个词也是知道意思，但是不会想到用，会需要LLM协助判断这是一个应该在口语/技术场景中广泛出现的词语吗？
- a rigid design system: rigid 来形容 system，这个组合也不会由我自主想得起来，而且 rigid 也是一个眼熟但是要说具体意思，我可能会去思考是严格？还是什么？
- **[01:40]** AI sucks at CSS.
- **[01:43]** It's a perfect storm, if you will, of things that AI does poorly.
- - 这句话： It's a perfect storm, if you will, of things that AI does poorly. 整体都没太明白什么意思，of things 没太明白这个 of 前面接的是什么？
- **[01:48]** AI loves to just solve a problem locally, instead of solving a problem globally.
- **[01:55]** It just likes to patch and throw things in there even when you explicitly tell it not to.
- - in there 这种也是口语频繁词，但是永远想不到自己去用
- **[02:00]** And with CSS, that often means just throwing just a bunch of bloated additional one-off classes or styled components or anything that you could be possibly in tailwind too.
- - one-off classes 不懂
- **[02:14]** It does the same thing with tailwind.
- **[02:16]** It could just throw a bunch of extra stuff.
- - throw off 这种动词，还是那种简单动词+off 之类小词的组合，很难记得住
- **[02:20]** Yeah.
- **[02:21]** Just jump pull up, jump, jump, and even like modern CSS as well.
- **[02:26]** Like yesterday I was working on it and I was
- **[02:29]** like use nesting and like it didn't use nesting at all.
- **[02:32]** And then it used the old version of nesting, you know, before we had like the, you don't have to put the ampersand in there.
- **[02:38]** And then like switched to bam at one point and then it doesn't know how to use any of the new features that are out there.
- switched to bam 什么意思？
- **[02:44]** And I know that these things can be solved with the right amount of prompting, but man.
- **[02:50]** And like that's not even what this guy is asking about, right?
- **[02:52]** Like, like, yeah, the CSS that it kicks out is that.
- **[02:55]** But also it also just
- **[02:58]** I don't wanna say awful, because like the like table stakes that like a cloud code will do kick out is like.
- table stakes
- kick out 简单词+小词的组合
- **[03:06]** like a black background, gray borders around a thing.
- **[03:09]** And it neatly puts them on the page and it looks decent, you know?
- **[03:13]** Like it doesn't look awful, but it doesn't look good in my opinion.
- **[03:17]** It's the layout, the information, architecture kind of sucks on it.
- **[03:21]** Quite a big task.
- **[03:22]** And I think a lot of people are trying to crack that nut right now.
- **[03:25]** Like Google released what stitch, Google Stitch the other day, where you can like basically it will come up with designs for you that are supposed to look way better than it.
- **[03:36]** I'm pretty sure they just have like 20 different templates.
- **[03:39]** Yeah, yeah.
- **[03:40]** That's how this works.
- **[03:41]** 18 different variables.
- **[03:42]** I think they're just simply feeding it.
- **[03:44]** A bunch of rules like always use this line spacing and whatnot, things that are known to look good as well as a bunch of like pre-made templates.
- **[03:51]** Because I'd like, I ran it a few times myself and then I was looking at what everybody else was generating on the internet.
- **[03:57]** And they all look...
- **[03:58]** similarly similar, which is also the tragedy of the columns with a lot of the stuff is it looks good Until everybody does it and then it starts to look like
- **[04:06]** Yeah, and I think some of it is system-pronzed stuff.
- **[04:09]** I would-
- **[04:10]** I got into it with my AI the other day where I was just...
- **[04:15]** yelling at it being like, why are you adding gradients here?
- **[04:20]** Why are you adding backroom?
- **[04:22]** Why are you doing this in that?
- **[04:23]** And it was like, you called me too.
- **[04:26]** And I said, I definitely did not tell you to.
- **[04:28]** I told you explicitly not to.
- **[04:30]** And it was like right here.
- **[04:31]** Here's where you told me to.
- **[04:33]** And it was a line of text I'd never seen before that I didn't write.
- **[04:37]** It wasn't in my agent's file.
- **[04:39]** So I googled that line of text and found it.
- **[04:42]** the open code.
- **[04:44]** So the codex header for open code says typography, use expressive purposeful fonts, and then it lists interroboto aerial.
- **[04:54]** Color and look, choose a clear visual define, avoid purple on white defaults, no purple bias.
- **[05:00]** The most egregious one here is don't rely on flat single color backgrounds, use gradients, shapes, patterns to build atmosphere.
- **[05:11]** And so I tweeted out, I said, I love open code, but this is not something.
- **[05:18]** harness like this should be deciding for me, right? These types of stylistic choices.
- **[05:23]** DAX responded saying that that is the system prompt from the Codex CLI and they just keep it as it.
- **[05:32]** That's why I think I'm moving a lot of stuff to pie because there's no system prompt stuff.
- **[05:38]** getting in the way. But even if there is this header text that gets put into all of your prompts.
- **[05:45]** It's also at a model level, these things are definitely biased into certain designs and certain styles, and they're trained on a bunch of garbage CSS.
- **[05:55]** I've tried to beat the purple out of these things and they've done a fairly good job of like beating the purple out of the model and like getting rid of it via prompts.
- **[06:04]** But now we're just sort of swinging so hard the other way where everything looks exactly the same as well.
- **[06:09]** You know, rounded corners, border on one side of a card.
- **[06:13]** I think they're also optimizing for normies that when they use it, it just looks good and they go, oh, AI is so good because amazing look good.
- **[06:21]** Instead of like, if you want practitioners with full control over it, now suddenly these things are gonna get in the way.
- **[06:28]** Man, yeah, it's a problem beyond like the design part of things and the actual implementation.
- **[06:35]** I'm trying to figure that part out West with my graffiti library.
- **[06:39]** This whole thing is built for the idea of like, here are templates, here is the customization.
- **[06:45]** Here's the things that you like.
- **[06:47]** Here is the structure and here's how it should work, but then you also, this is how you customize it and you don't add, you don't add one-off classes to do this thing, you use the classes for the structure and then you build on top of that systemically.
- **[07:01]** And I've like built a best practices skill.
- **[07:03]** I've tuned the docs now, the docs return, mark down, we'll talk about that later, but like I've just been grinding on this specific problem for a while now because I hate this thing, yeah.
- **[07:16]** I hate this.
- **[07:17]** It's honestly a very hard problem to solve because like in many other cases, you probably will want to step outside of some sort of design system because you just know.
- **[07:25]** And I think the answer, at least right now, to a lot of the stuff is like, you still need humans with talent and good taste, right?
- **[07:32]** I know everyone hates the word taste right now, but...
- **[07:35]** a lot of
- **[07:37]** doing good design and like laying things out so it makes sense and not just adding buttons absolutely everywhere.
- **[07:44]** which just comes down to somebody that actually knows what they're doing and can be powerful with these tools, right?
- **[07:51]** Yeah, totally.
- **[07:52]** I don't think anybody, like I've sort of try and low for this type of stuff.
- **[07:57]** And unless you have...
- **[07:59]** I think if you have a very rigid, well-documented design system, that certainly helps quite a bit.
- **[08:06]** But still, I don't know anybody that has certainly nailed this just yet.
- **[08:10]** Same.
- **[08:12]** And like the people...
- **[08:13]** I got into it with some knucklehead on Twitter the other day where like he just says like
- **[08:20]** I was amazing at design all you need is a screenshot and he says great artists make or great artist steel whatever that quote is You know and then I like just like screenshot at his website It's like you're you're spouting this BS off in your own website looks like something clawed just
- **[08:37]** it out in like 12 seconds. And you think it looks good because you got an taste. Yeah. No, but that's the thing that aside from that, the guy was selling like an accessibility tool for checking color contrast and literally his own website didn't use the tool and didn't what didn't have the contrast in it.
- **[08:58]** That drives me nuts because people are probably in the comments already to this one being like, oh, I can do X, Y, and Z. And I have a great thing. And it's like, no, your website looks like the same vibe coded app that everybody else came out with. You know, I've never never been impressed or enjoyed using design of one of these vibe coded things. It certainly is better than a lot of the crap people were
- **[09:21]** like just like cludging together themselves, which is good, means like the baseline for like decent experiences is much higher now, but I don't think that you can simply just screenshot a good design of some other app and say, oh, mine now looks amazing as well.
- **[09:37]** Yeah, I think this all comes back, I had a tweet that was, it's crazy how AI is really good at the stuff I don't know anything about and total dogs at the stuff I do.
- **[09:47]** Because the moment that you have an eye for CSS and eye for design and eye for anything, the slop looks like slop.
- **[09:55]** And there's a reason why I think my AI code for backend stuff is great, because I don't, I'm not as good at that stuff, I'm good at backend, but I'm not like an expert.
- **[10:06]** So, hey, it works, it works, but when I look at anything closely, especially front end code, the stuff that I'm good at, the design stuff I just,
- **[10:15]** I see how terrible it can be.
- **[10:17]** So yeah.
- **[10:18]** Yeah.
- **[10:19]** I think that I certainly much better at backend code, which is very deterministic, very programmatic, following patterns.
- **[10:26]** And then when there's something that's a lot more visual and has to come into life.
- **[10:30]** the taste that you have.
- **[10:31]** It's much harder.
- **[10:32]** Not to say, I think we probably will crack this.
- **[10:35]** I think somebody that actually knows it.
- **[10:36]** I know Tailwind guys are working on UI to SH right now, and I've seen some decent stuff.
- **[10:41]** They've been cranking out.
- **[10:42]** A lot of smart people working on this right now, and I bet we'll probably see it solved in the next six months or so.



- **[10:50]** Lewis says, I'm getting back into hands of web development after years of working in product roles.
- **[10:56]** I want to get my hands dirty again.
- **[10:58]** So I'm doing a React app to prove also to myself that I can still do it.
- **[11:02]** But it is so much easier to get ahead to write most of the code myself.
- **[11:07]** This is a dilemma for me and I am also learning.
- **[11:10]** Since I am also a teacher, I know I am learning less when I use AI, but then again, maybe some of the skills I am missing out on here are redundant and I'm better off learning how to code with AI on my second screen.
- **[11:23]** He goes on to say, I do know the fundamentals of coding with React.
- **[11:27]** I just haven't been writing lots of code myself the last couple of years.
- **[11:30]** What are your takes on this?
- **[11:31]** Yes.
- **[11:33]** This is kind of an interesting one.
- **[11:35]** And I think as much as people...
- **[11:38]** I don't know, it's a delicate balance here.
- **[11:41]** I certainly think you still do need to understand how the ins and outs of a lot of this stuff work, but I don't think you need to know the ins and outs of it as much as you previously did.
- **[11:52]** I think that people that are building the lower level stuff of this certainly need to know how this works, but that they'll be then turning that into libraries and whatever that we can sort of sit on top of.
- **[12:03]** And then the people that do know how a lot of this bad stuff happens, like whether it's performance or optimizations, they will be creating skills that can be modeled out and you can apply it to your own code base.
- **[12:15]** So like what's my take on like do you still need to know how all of the stuff works?
- **[12:21]** I think the people that are getting the most out of
- **[12:24]** this AI right now are the people that actually understand.
- **[12:29]** how to tackle large projects, how technology works, how these things all work together.
- **[12:34]** You see Toby.
- **[12:36]** from Shopify, obviously creator of Shopify.
- **[12:39]** That guy hasn't slung code in many, many years and then all of a sudden he's just...
- **[12:44]** these really interesting projects, you know, QMD and auto research, pi and whatnot.
- **[12:51]** And that's because...
- **[12:53]** of the type of person he is.
- **[12:55]** He's a thinker, he's a technologist, he understands planning products, he understands sort of the bigger picture of a lot of this stuff.
- **[13:03]** And you're starting to see a lot of like, really reputable project managers, I know that term gets, not to say Toby's a project manager, but you're seeing a lot of people who previously were sort of hands off with the code and much more of a skip sheer error.
- **[13:18]** Ship steerer, now they're sort of getting back into the code.
- **[13:22]** And the stuff they're building is good because they...
- **[13:25]** they understand how all of this stuff works.
- **[13:27]** So that's my take there is that.
- **[13:30]** The people that are going to do the best on this are not just the people that can type in the box and produce a bunch of garbage, but the people that really know how to think about a project and apply, solve problems, you know, all of the things that were at the end of the day were using this code for.
- **[13:46]** From a ship, steerer to stirr.
- **[13:50]** I, you know what?
- **[13:52]** It's so funny, Wes.
- **[13:53]** I agree.
- **[13:54]** I think one thing that could help here is just the idea of
- **[13:59]** slowing down a little bit and reading the code.
- **[14:03]** I know it's when you're learning something, there's a little bit of a difference there because you're not going to be able to recognize bad patterns, but with the amount that AI does write kind of...
- **[14:14]** lot of patterns. If you can read the code that it's writing and identify where the problems are, or even ask it to review its code and see possibly if it has ideas for where these sloppy things are, I think there is a loop there in which you're able to pick things up that you wouldn't have picked up before by really having a deep eye for reviewing the code rather than if you're enjoying having the eye right the code. But being able to have an eye for what is good and what is bad, I know that's tough to do without...
- **[14:53]** really writing the code for a long time, many of the patterns that I can recognize as being bad, I only recognize as being bad because I wrote them poorly myself at some point and then refactored.
- **[15:08]** Well, I'll give you another example of where it matters about the code, but so much more the actual implementation.
- **[15:16]** So I have this one Photoshop that's called texture anything.
- **[15:20]** It's a Photoshop plugin called texture anything.
- **[15:22]** And the way it works is that you
- **[15:25]** you drag this plugin in and it has a list of, I don't know, like 30 or 40 Photoshop actions that it applies to it and then it will sort of give like a logo, like a texture, grunge look to anything.
- **[15:38]** We'll put some screenshots up if you're watching the video right now.
- **[15:41]** And I love that plugin, but I just like, I hate having to open Photoshop for it or it takes like a long time and I was like, I would love for this to be programmatic.
- **[15:49]** So I got Claude, I dropped it in Claude, I said convert this to using JavaScript, right?
- **[15:55]** And then it went through it.
- **[15:56]** It surprisingly was able to read the Photoshop action, figured out what each action was doing, and then out the other end, boom, it gave me the entire effect done in, I think it was in sharp, and it did a great job.
- **[16:10]** Like it looked exactly what I wanted, right?
- **[16:12]** It re-implemented all of the blurs, all of the, every single thing that Photoshop would do, it now did it programmatically in code.
- **[16:18]** But the problem was it was slow, right?
- **[16:20]** Like if I uploaded it to 500 by 500 pixel syntax logo, it would take 10 seconds to apply it.
- **[16:27]** And I was like, I would like for this to be much faster because there's knobs I would like to turn.
- **[16:31]** You know what happens when you turn the green level up, and you got to wait 10 seconds.
- **[16:35]** So I like.
- **[16:36]** like my programmer brain and I was like, how are you doing this?
- **[16:42]** You know, what technologies are you using?
- **[16:44]** And over the course of maybe two hours, I sort of went back and forth and said, why don't we use wasm for this?
- **[16:50]** And then, okay, oh, good idea, you're so smart.
- **[16:52]** I know, I think, you know?
- **[16:54]** Let me move this over to wasm and then I was like, we'll like,
- **[16:57]** how long each step takes, right?
- **[16:59]** And then I would look at it and say, oh, this step is taking much longer.
- **[17:04]** Like, what are you doing with this?
- **[17:05]** Oh, I'm looping it through every pixel, but we don't really need to be looping over through every pixel.
- **[17:10]** We just need to be, maybe we can filter four pixels that have a color first, you know?
- **[17:14]** And then I got it down to like 200 milliseconds, you know?
- **[17:18]** 10 seconds to 200 milliseconds, all because I knew how like graphics processing and canvas works, I knew how web assembly works.
- **[17:26]** I knew how to parallelize things in web assembly.
- **[17:31]** And I didn't write any of the code myself, but like, that was a major.
- **[17:34]** If I was just to type it, like make it faster, maybe it could figure it out.
- **[17:38]** And maybe that's something for I could use like a auto research plugin for, but...
- **[17:43]** I think most people would have been like, check this out, I made this amazing.
- **[17:48]** Photoshop to JavaScript converter and it works at the end of the day and it's like well
- **[17:54]** Yeah, it works, but it's not really that great of an implementation.
- **[17:58]** And I think the code that I kicked out at the end of the day was really good and it's way faster.
- **[18:04]** We have an episode that I want to work on.
- **[18:06]** Well, I've already wrote most of the episode, but
- **[18:09]** using more deterministic methods for improving the AI output, whether that is quality checks or speed checks or those types of things, where it's not just telling the AI, hey, go make it faster, because it will take shortcuts or cheap out or a lie or do all kinds of stuff for you.
- **[18:28]** So I think there's a lot here in terms of being able to understand where the problems are being knowledgeable enough to direct the AI and away to fix these problems. Again, I think those are all interesting questions.
- **[18:41]** Alright, next question here is from Alex to preface my question on the network engineer and don't have much web developer background, oddly enough.
- **[18:49]** I enjoy the syntax podcast despite having very little context to keep up with some of the episode.
- **[18:55]** With that said, I've been a part of a fair share of company website performance investigations, where I typically turn to the browser developer tools network tab and confirm supporting infrastructure is operating normally to contribute.
- **[19:11]** I'm looking for other ways to bring greater value to these investigations from an IT operations infrastructure perspective, what insights tools or practices would be helpful to web teams during performance troubleshooting.
- **[19:27]** Yeah.
- **[19:28]** Yeah, so Alex, this is great.
- **[19:31]** One thing I will say here is make sure that the teams you're supporting with this stuff.
- **[19:36]** would like your support.
- **[19:38]** Not because, not because that you're not being helpful, but sometimes it can be not helpful when somebody who isn't having like the full picture is giving.
- **[19:49]** be back on stuff, you know, sometimes it might even be better to be more helpful to say like...
- **[19:56]** I'm noticing these pages are slower whatever rather than trying to give a deep
- **[20:01]** like technical explanation as to why the worst thing is, is that when somebody doesn't know what they're talking about, but also...
- **[20:09]** punches something into Claude or chat you be T and then paste the reply to you.
- **[20:13]** Yeah.
- **[20:15]** That's so disrespectful to me, because like, you're not suggesting anything.
- **[20:19]** I could have punched that in myself as well.
- **[20:21]** And it's your way, see my time, making me read this now, as if you came up with...
- **[20:25]** Anyways, continue.
- **[20:26]** No, yeah, no, I think that's right along the lines of it.
- **[20:30]** We're like, I think it is helpful, Alex, if they find what you're doing to be helpful.
- **[20:35]** That said, we do have a number of episodes where we've talked about this in Go deep in depth.
- **[20:39]** Episode 585, fundamentals, what makes a website slow?
- **[20:44]** We also talk about it in 8.74, fast apps, easy perf wins.
- **[20:49]** We talk about an 8.97, making your apps feel faster, then it really is.
- **[20:55]** And 9.72, these things make your app feel like crap on mobile.
- **[20:59]** These episodes are all a great place to get dive into a little bit of what exactly you should be looking for.
- **[21:06]** But again, most websites, I will say, you know, maybe it's a little bit different in the modern era, but fundamentals are still here.
- **[21:14]** Most apps are slow because they're loading heavy resources.
- **[21:19]** The Network tab is a great place to see what is taking a long time.
- **[21:24]** Like you said, it can identify if the server response to slow, is it loading some giant images or something where it shouldn't be?
- **[21:32]** There's a number of things you can certainly learn from the Network tab there as well.
- **[21:38]** Yeah.
- **[21:38]** He was saying like, he's a Network Engineer and he knows how to open the Network tab.
- **[21:44]** And that's a pretty good spot to start.
- **[21:47]** But like, there's so many other.
- **[21:49]** spots in the like from a website being a baby to actually be delivered to your your browser like that can Introduce latency or or slowness ones right like if you if you the network engineer realize oh
- **[22:03]** 400, 800 milliseconds is taking for this page to actually respond from the server, even just to get the HTML.
- **[22:10]** At that point,
- **[22:12]** You're just like well that slow but like like but why you know and at that point you probably have to do
- **[22:18]** start to figure out how the app actually works.
- **[22:22]** And as a network engineer, you probably have some good insights into life.
- **[22:27]** how to make this thing faster.
- **[22:29]** And you get some observability in there.
- **[22:32]** You get the metric.
- **[22:33]** So what's actually taking, is it database?
- **[22:35]** Is there something like weird thing that we're waiting for?
- **[22:38]** The tweets to load before we send the whole thing out.
- **[22:41]** And that's taking 400 milliseconds and holding the page.
- **[22:45]** Pass that, understand caching, look into StaleWare, Revalidate.
- **[22:48]** Those things can certainly help quite a bit.
- **[22:51]** Also, like we said, there's a lot you can do on the UI layer to make your website feel faster.
- **[22:57]** In some cases, the actual rendering is slow, and that could be...
- **[23:02]** how your resources are being loaded in, or it also could just be like, you are re-rendering these things way too often.
- **[23:09]** That can sometimes happen in React applications.
- **[23:12]** So just like kind of go through all four of those episodes and just understand where slowness can be introduced.
- **[23:20]** And when you're on the website, does it feel slow?
- **[23:23]** Is it actually slow?
- **[23:24]** Understanding all of these tools.
- **[23:26]** And like the network tab is not just the only one you can go into like the rendering tab, the flame graphs, understand all of that stuff.
- **[23:32]** I think that's a...
- **[23:33]** That's going to be very...
- **[23:35]** good skill to have, I think, in the next little while as everybody's just cranking out these websites where they don't necessarily know what the code looks like.
- **[23:45]** Being able to just dive in and figure out, all right, well, we have this thing, but like, how can we improve it?
- **[23:51]** Where is it actually going slow?
- **[23:52]** Yeah, I will say here's an opportunity to plug Century SEMTRY.io.
- **[23:59]** One of the coolest things that Century does is they have tools that can identify what your slowest routes are or potentially what your slowest DB queries are, those types of things.
- **[24:12]** They can make it really easy to find exactly where the slowdowns are in any of your applications.
- **[24:18]** Man, we use this so, so much when we built the current version of the syntax site.
- **[24:23]** I'm sure we'll use it just as much when we finally finished the next iteration of the syntax website.
- **[24:30]** But it can really give you a misery score and stuff for how...
- **[24:36]** frequently this page is hit versus how slow it is.
- **[24:40]** There's a lot of great stuff in Century for tracking these down as well as really getting into where these slow downs might be.
- **[24:48]** Yeah, the front end has like the web vitals, performance scores, all that kind of stuff.
- **[24:54]** The database one we had, we've told a story many times, but I think it's hilarious, is that CentriSent is an email one day, like as an alert, being like your database queries are climbing and they're at a point where they're too slow now.
- **[25:06]** And it turned out we didn't have an index on our shows page.
- **[25:09]** So every time we added another show,
- **[25:12]** it just increased the size of the database.
- **[25:14]** And when we were querying based on, I think we were querying based on the show number or something like that.
- **[25:19]** And there was no index.
- **[25:20]** So it had to loop through every single show in the database and look for it or looking for the, I think the slug, something like that.
- **[25:28]** And like,
- **[25:29]** I don't know that we would have noticed that because it got just a couple of milliseconds slower every single time we pushed a show, right?
- **[25:36]** And that's not significant.
- **[25:38]** But over adding 400 shows since we built it.
- **[25:41]** It got way slower and then we're like, oh, pop an index on that.
- **[25:47]** And then we were good to go, which is hilarious.
- **[25:49]** Yeah. Well, you know what?
- **[25:51]** I'm seeing right now that SinHax is getting some not great scores for web vitals.
- **[25:58]** I'm wondering why.
- **[25:59]** So I need to dive into that.
- **[26:00]** Considering it is using the very fast zero sync and in practice, it feels fairly fast, but I'm sure there's some stuff to learn here about why we're getting slow performance scores on here.
- **[26:10]** interesting. So check it out, century scntry.py.io, ford's lesson tech, sign up, use the coupon code, tctrilur, case all one word. These are some great tools.
- **[26:19]** And I think very relevant to this question. Sorry, I didn't mean to turn your your question into an ad Alex. It just happened to work that way.
- **[26:27]** Mike says, hi guys for Sephormus, I want to say thanks and keep your flowers.
- **[26:33]** I was able to move into a developer role because of the podcast in both of your tutorials.
- **[26:37]** Well you're welcome.
- **[26:38]** I want to get into fixing electronics as a hobby, Wes.
- **[26:42]** What do you recommend as a good setup?
- **[26:45]** or a soldering station for a beginner.
- **[26:47]** Yes, man, I've got, I've got thoughts on this.
- **[26:50]** S soldering iron, you can, you can pretty much just get like anything, but if you want to get something like decent and cheap, I have the RyoB.
- **[26:59]** battery-powered soldering iron.
- **[27:02]** And then I also have the little, I think it's called like a T100.
- **[27:06]** It's like a USB little pen shape soldering iron.
- **[27:09]** And I do reach for the Ryobi one much more often because when I reach for the USB-C one, I can never find the frickin power cord that is 100 watts or I can't find like a, something to plug into the wall.
- **[27:24]** And I much rather just have the thing on my desk.
- **[27:29]** So I can solder with it.
- **[27:30]** So big fan of those, it doesn't have to be the Ryobi one, can be literally any battery powered one.
- **[27:37]** So whatever tool brand you have, I think even Adam Savage.
- **[27:40]** has a self-build one where he took a DeWalt battery and he plugged one of those little the barrel adapter ones right into it.
- **[27:48]** So that's my recommendation for there.
- **[27:50]** Along with that, I would probably recommend getting just a whole bunch of little connectors and clips and whatnot, or even just take apart.
- **[28:00]** I figure it's called like a solder sucker where you push it in and you let go and go
- **[28:05]** and that what that does is it sucks the solder out.
- **[28:08]** I'm trying to see if I have it on my desk right now.
- **[28:10]** Now I cleaned up my desk.
- **[28:12]** A little bit of flux in like a little syringe.
- **[28:15]** Flux is really messy.
- **[28:17]** And you need it when you're soldering or desodering components.
- **[28:20]** So I always recommend getting one in like a little squeezy pen.
- **[28:23]** So you don't just get flux, make a mess.
- **[28:25]** Flux is, when you're heating up your solder, it doesn't want to nicely flow into it.
- **[28:32]** So a lot of times you see people that are like, oh, I suck at soldering, look at it.
- **[28:35]** And it's just like this like ball of crustiness.
- **[28:38]** That's me.
- **[28:39]** And that's, yeah, well, it's probably because you don't have flux.
- **[28:41]** And it flux is like this like gooey paste that you put on top of it.
- **[28:47]** And when you apply heat to it, it's like,
- **[28:49]** does something to, I don't even know how it works, but it basically makes your solder just kind of flow out.
- **[28:56]** And you need that when you want it to like flow into a joint that you're soldering, or the opposite, if you want to like suck up the solder with like a copper wick or something, you need to put a little bit of flux on top of it.
- **[29:10]** So I need some flux and some solder.
- **[29:13]** Yes, absolutely.
- **[29:14]** And like a little, what's it called?
- **[29:16]** Like a braid?
- **[29:16]** I don't know the names of any of these things, but you just Google it.
- **[29:19]** finds us at GoOnAllyExpress.
- **[29:22]** So take a look around for all the fun things that you can buy, figure out what the absolute cheapest thing is and then buy like one level up from that you'll probably be happy.
- **[29:31]** That's good advice.
- **[29:32]** Although, yeah.
- **[29:33]** Be careful with the light.
- **[29:34]** lead solder is certainly a thing in China, which a lot of people love lead solder because it's so much easier to work with, but also it is...
- **[29:43]** very harmful to your health. So if you are buying stuff off of valiExpress, be careful that you're not actually buying lead if you're going to be huffing it in.
- **[29:51]** Yes.
- **[29:52]** Yes.
- **[29:53]** Important.
- **[29:54]** All right, next question from Kaelin.
- **[29:55]** Hi, Syntax.
- **[29:56]** How do we balance prepping for NOAAI interviews while job descriptions demand AI profitions?
- **[30:05]** Honestly, after getting spoiled by tab completion and instant AI answers, manual coding feels like going backward.
- **[30:13]** is old school prep still the only way to prove experience in 2026.
- **[30:20]** Well, yeah, that's an interesting thing to have a job that demand...
- **[30:25]** AI proficiency, but doesn't allow you to use AI in interviews feels like...
- **[30:31]** bad structure for an interview if you're asking me.
- **[30:34]** Like, this job demands jackhammer proficiency, but guess what?
- **[30:40]** You gotta just use a chisel.
- **[30:42]** Like, okay, what are you testing me here?
- **[30:45]** I think some of this too is like really just kind of dialing in your...
- **[30:50]** your understanding of things because many times, like the code we write is like less important than understanding the concepts of writing good code.
- **[31:04]** So a lot of this too.
- **[31:06]** is less about the code you end up writing and more about like being able to express the understanding of systems to express the understanding of best practices.
- **[31:19]** So even if you're used to the tab completion style of things, if you can prove to them that you understand the core concepts, you understand why you're doing things and what these best practices are, I feel like those are the things that are going to always keep you ahead of somebody who's just like, I don't know, I just hit the button and it works.
- **[31:43]** Or I'm putting this component in.
- **[31:45]** Why am I putting this component in?
- **[31:46]** I don't know, it's just how I've always done it.
- **[31:48]** You're being able to express yourself and show your work in those types of ways.
- **[31:53]** To me, I think lands better.
- **[31:55]** But it is interesting that they wouldn't allow you to use AI in a tool that demands, or a job interview that demands AI proficiency.
- **[32:03]** I saw this from Brendan Falk on Twitter the other day and I thought this was genius.
- **[32:08]** He says, I believe we have found the best AI native coding interview.
- **[32:12]** We call it the Composer One interview.
- **[32:14]** Composer One was like the very first model from cursor.
- **[32:17]** And...
- **[32:18]** He says candidates get one hour to build a real medium-sized project live.
- **[32:23]** The only constraint is they must use Kirscher's Composer One Model.
- **[32:26]** Why Composer One?
- **[32:28]** It's extremely fast.
- **[32:29]** Candidates can accomplish a lot in one hour.
- **[32:32]** More prompts and more code equals more surface area to assess.
- **[32:35]** And two, it's perfectly mediocre.
- **[32:38]** I love this.
- **[32:38]** Composer One is good enough.
- **[32:40]** The output is decent, but not good enough that it does everything perfectly.
- **[32:44]** Candidates can't rely on the model.
- **[32:46]** They have to be good engineers themselves.
- **[32:48]** We candidates use one agent, write bad prompts, and accept the code they don't understand, and end up with poorly structured codebase.
- **[32:56]** Exceptional candidates run parallel agent.
- **[33:00]** right detailed prompts and enforce very high coding centers.
- **[33:04]** I saw this, I was like, this is genius.
- **[33:06]** Like.
- **[33:08]** Give them something that's not that good, but is fast.
- **[33:12]** And see how they easy, so this is not really answering your question about like, how do you prep for a no AI interview, but I thought it was just so smart.
- **[33:21]** Oh, yeah, this really separates people who understand how these systems work, how they can plan, how they can tackle things versus like people are just like, I don't, I just pushed the button.
- **[33:31]** I typed in the box what you told me to build.
- **[33:33]** And I'm gonna accept everything that comes in.
- **[33:35]** And that goes along with what I was saying is because if the, even if it's tab completion and not like prompts and agents, even if it's just tab completion, and the tab completion is giving you something that is bad, you learn a lot from watching somebody modify the tab completion or choosing not to use the chosen tab completion because of why it's, why it's been.
- **[33:58]** or learned a lot by watching you not accept your tab completions as well. That's how, for sure, that's how they trained all their new model, their composer too. They have all this data on, they have all this data on what people have accepted, right? They're sitting on this gold mine of we have done two billion whatever requests and we propose this code. Here's the cases where people just deleted the whole thing. Here are the cases where people change these things. That's pretty good.
- **[34:27]** The cursor rolled out their composer too a couple days ago and it actually is pretty good. But you saw the drama around that, right? Yeah, yeah. So they took what they took Kimmy, 2.5 and then trained on top of that. But they didn't say that that's what it was. So there's all this drama around.
- **[34:46]** people thought they just took the, the Kemi 2.5 slap their name on it and called it a day, which is a bit of a PR crisis for them.
- **[34:55]** Yeah, a little bit.
- **[34:56]** Yeah.
- **[34:57]** But that said, it's actually really good.
- **[34:59]** I used it for like a week in it.
- **[35:00]** I was like, this is actually pretty good.
- **[35:02]** It's not as good as, as like Opus or Codex or any of that, whatever the latest Open AI one is, but...
- **[35:09]** Pretty good, I would say.
- **[35:12]** Pretty good and you can use it with Zed.
- **[35:14]** I didn't think you could but you can.
- **[35:16]** Air.
- **[35:17]** He says, have you tried Figma?
- **[35:19]** What do you think about their Dev Mode?
- **[35:20]** So yeah, I thought this was an interesting question since we're talking about design and how to approach these types of things.
- **[35:28]** I've never used Dev Mode in Figma.
- **[35:30]** Have you used God?
- **[35:31]** No, no.
- **[35:32]** Get that out of the way first.
- **[35:33]** No, I've been, I've looked at it, but I've never considered using it for any serious anything.
- **[35:40]** Yeah.
- **[35:40]** Yeah, I always look at these types of things and I'm like, I don't really want.
- **[35:46]** my design tool writing code for me.
- **[35:49]** And now we're at a spot where...
- **[35:52]** you can just screenshot the thing, to show it out to one of the last thing and just have the thing coded for you.
- **[36:00]** I don't know that ways necessarily need that.
- **[36:02]** I will tell you that what we do need, we still need an app.
- **[36:06]** that is like Figma to actually design these things.
- **[36:10]** And whether you're doing that via prompting and it's just kicking out code, which is what a lot of like I know paper and then I think there's another one called pencil, there's several of these apps people are working on, which is just like...
- **[36:23]** You have to pick mypot instead of...
- **[36:26]** instead of it rendering out like WebGL or whatever Figma is using, it's literally just rendering out HTML elements to the thing.
- **[36:33]** We certainly still do need apps like this that are going to help us figure out what the design looks like.
- **[36:39]** And some sort of interface, whether that's clicking around and dragging things or whether that's inputting values into boxes or whether that's simply just typing into a prompt box and saying, space these things out 20 pixels a little bit more and seeing what the result is.
- **[36:53]** I think we still need those apps because the alternative
- **[36:58]** just yolowing it and having, this is what we talked about earlier, did the alternative of just yolowing it and having Claude kick out a design for you is not working out.
- **[37:08]** And it's making some pretty brutal interfaces.
- **[37:12]** I've been actually going into Figma lately to design instead of designing in browser, but not using the code mode or code output, or even giving, I'm not even giving screenshots of it to AI.
- **[37:24]** I've just been really liking Figma lately for just straight up designing instead of, I primarily design in browser, but as Wes knows more than anybody, what happens with me is I start from such a systemized point of view that when I design in browser, my designs feel soulless because I don't get chance to play, I build the system, and then okay, it looks fine.
- **[37:49]** I'm bored, let's move on.
- **[37:51]** Where if I go into Figma, I can really get Arty with it and throw stuff at the wall.
- **[37:56]** I actually really like some of the auto layout stuff in Figma, but they're dev mode, they're code stuff, I personally wouldn't touch it, because I don't trust somebody else's code.
- **[38:06]** Except for a clock.
- **[38:09]** I don't trust Clod's code, I'll tell you that.
- **[38:11]** Okay.
- **[38:12]** Yeah.
- **[38:12]** I did test, what was it?
- **[38:14]** Paper the other day, where the idea is that paper is like a canvas where you can like feed it.
- **[38:21]** website or like a component and then it will recreate that component in
- **[38:27]** paper and then you have this like inner tape
- **[38:31]** which you can either click and drag and around, like visually, or you can also just prompt with an MCP server to change things.
- **[38:40]** And then the idea is that you then, once you're happy with it, you put it back into code.
- **[38:44]** And your code base is sort of the source of truth.
- **[38:47]** Only when you need to like work on a design, you sort of take a piece of the website, out of the website into this, like what do you call it, like Canvas, you work on it with your designers and whatnot.
- **[38:58]** And then once you're happy with it, you stick it back into the website.
- **[39:01]** So that was kind of interesting way to go about it.
- **[39:05]** So you don't have this like designs, and then you have your actual implementation of your website where they start to drift from each other.
- **[39:12]** Yeah, so again, I don't know that we know what the answer to all of this stuff is, but certainly a lot of people are trying to figure out what that looks like.
- **[39:20]** Next one here from Bosch, not really a question, but West, your sick pick the other day was ICE, which is inactive and you should use law instead.
- **[39:27]** Yeah.
- **[39:28]** You thought we'll link up the thoughts, the modern version of thought of ice, which is a menu bar manager in Mac OS and thought is pretty much a straightforward that's being maintained of ice.
- **[39:42]** And I gotta say, I've been using thought for a bit.
- **[39:44]** And, hey.
- **[39:47]** That looks good.
- **[39:48]** I have had zero issues with my ice, but I didn't even realize it was un-maintained.
- **[39:52]** I, you will, I'll tell you, you'll have issues when you upgrade to the latest version of macOS because I'm on the betas and it doesn't work.
- **[39:59]** And then I install Baw and it works just fine.
- **[40:01]** So, it's good.
- **[40:03]** Yeah, I'll tell you what it should have.
- **[40:06]** I have with it is I use ice to add little backgrounds to my menu bars.
- **[40:11]** It's kind of cool and it looks nice.
- **[40:14]** And they're a little bit laggy in how they draw.
- **[40:18]** Like if I hover over top of some of the items, you can see that they don't paint as quickly as possible.
- **[40:24]** So maybe that's fixed.
- **[40:25]** I'll give it a shot.
- **[40:26]** Thank you, Bob.
- **[40:27]** Mathman Sense says since AI has made writing code and changing so much easier, one would expect as a collective, we would drift towards more performance or more stabilized APIs.
- **[40:39]** For example, moving servers to go or moving front end code to web standards and web components instead.
- **[40:45]** But I have seen a bigger shift towards proprietary ecosystems like next JS.
- **[40:52]** What do you think that is if you accept my promise at all?
- **[40:54]** Yes, lots of things to say here.
- **[40:55]** First of all, we are recording an episode.
- **[40:58]** I think by time you're listening to probably be out a week before this one on moving systems, right?
- **[41:03]** Like I move my express code base, which has been bugging me for many years.
- **[41:07]** I moved over to Hano and we have a really good episode on like how to tackle a project like that because I don't think it's as simple as like move my server to go, and then all of a sudden it just moves absolutely everything over and it's totally fine or make everything fast as possible.
- **[41:25]** It don't think where they are just yet with this type of stuff.
- **[41:28]** And.
- **[41:29]** The question here is, I've seen a bigger shift towards proprietary ecosystems like next JS.
- **[41:35]** Sure, by time you listen to this announcement on the next JS stuff will go live.
- **[41:39]** They are rolling out a whole bunch of...
- **[41:42]** stuff about hosting elsewhere and adapters, which is pretty cool.
- **[41:46]** So we're going to be talking to them about that as well.
- **[41:49]** What do you think that is?
- **[41:50]** I don't know.
- **[41:51]** I wouldn't say these things are proprietary.
- **[41:53]** If anything, like a lot of these like...
- **[41:57]** That's how Linktree the other day had jacked their price up by three X or something stupid like that.
- **[42:04]** I'm like, yeah, that's because you shouldn't pay somebody.
- **[42:07]** or a page that has a six links on it.
- **[42:10]** You know, that's ridiculous that you are even doing that in the first place.
- **[42:16]** and...
- **[42:17]** people are realizing, oh, I don't need to pay $300 for a link tree.
- **[42:21]** I can simply just make my own.
- **[42:23]** So I wouldn't say we're moving towards proprietary stuff, not at all. I think we're moving much more standard space much more
- **[42:32]** of this stuff that is being built today. If you look at what the Claude code stack is, you know, it's all very much based on web requests, web response, fetch, streaming, all that good stuff.
- **[42:43]** Yeah.
- **[42:45]** I do think there is this like default AI stack that if you tell it you're building a web app at leans on all this stuff that you don't...
- **[42:53]** need often times.
- **[42:55]** Yeah, I wonder the same thing, Moth, man.
- **[42:57]** I've been doing some rust with AI, and it's really nice.
- **[43:00]** I've been doing all kinds of stuff that, again, I'm not like, yeah.
- **[43:06]** I'm sorry, I don't have a formed opinion on this even though I've just been thinking really hard about it And I would think it really hard on I feel like I got an eloquent answer here And then it comes out like that Uh, Mothnet I think it's just what it's trained on and what it's like really steered towards We looked at that in the state of JS where it's like this is the stuff that AI is gonna suggest the most And I have a hard time with that because it's always suggesting a bunch of stuff
- **[43:34]** like I don't want it to use, it's never, I never had it ever once suggest to me to use web components, even though there are times when a web component would be the right call there.
- **[43:44]** Yeah, I think some of it is all comes down to, we're gonna say it again, personal taste.
- **[43:49]** And you have to know what you want.
- **[43:50]** There's your competitive advantage.
- **[43:53]** Yeah, like knowing what you want.
- **[43:56]** I don't know, the people that are complaining about AI not-
- **[44:00]** suggesting what they want.
- **[44:02]** Like, are we that lazy?
- **[44:04]** that you can't spend three minutes.
- **[44:07]** figuring out what framework you want to use and you're just you're winding and like not not directly at you moth man Yeah, like man you won't man you're cool. You're one of us, but like come on are we that lazy that like we have to like worry about this type of stuff like let the people I guess like if those are your competitors that are not using go as a server And it's it's really gonna slow you it slow them down so much then then type up a new one, you know competitor making new
- **[44:37]** I'm a lot for something.
- **[44:38]** Yeah.
- **[44:39]** Yeah.
- **[44:40]** A lot there.
- **[44:41]** I don't know.
- **[44:42]** I think the whole theme of this episode is like, you still have to think.
- **[44:46]** Please, please, but like you still have to, you can't be an idiot yet.
- **[44:50]** You can't, yeah.
- **[44:52]** Use your brain.
- **[44:53]** Yeah.
- **[44:54]** Use your brain, folks.
- **[44:55]** All right.
- **[44:56]** Jim Carrey clone says, hey, Wes and Scott, while referring to tan stack router docs, I mentioned something called vibe rules.
- **[45:02]** And after digging, it seems like an NPM package, NPM packages, example, tan stack, forward slash record, or now ships an LLM folder inside the disk of the node modules, which can be installed with vibe rules.
- **[45:17]** Is this replacement for Context 7 or Agent Skills or at least Docs have you used it?
- **[45:24]** Personally, Scott, have you used Vib rules?
- **[45:27]** Yeah, so Vib rules is a package and what it does is allows you to manage rules file.
- **[45:37]** So whether that's like cursor rules or those types of things.
- **[45:41]** Now, the way that this works is through a CLI where you would run vibe rules install and you can point it to things.
- **[45:50]** And the idea would be that you are shipping the MD with the library.
- **[45:54]** So that way you could install the vibe rules into your whatever it is you're using just via this CLI because it's been shipped.
- **[46:04]** I don't really use the style of like rules files.
- **[46:09]** Now it could put it into it looks like it could put it into the agents dot MD file.
- **[46:14]** It can put it into a claw dot MD.
- **[46:17]** There's some of this that I can understand existing, but I would much rather it ship like at this point personally, I would rather it ship a skill.
- **[46:31]** and then just have that skill available rather than having it be a rules file or, I don't know, I hadn't seen vibe rules before we got this question in, and it doesn't necessarily feel.
- **[46:45]** like something that I'm personally going to be, like going nuts to use, but it does, like the one thing I do on a plot, the tan stick folks here is taking ownership of that part of things because too often times we are just pointing the AI at llms.txt or we're crafting our own custom skill or our own custom agents file.
- **[47:09]** So if the library can tune one up specifically, then that's great.
- **[47:14]** I've been working a little bit this I mentioned before that I have like a graffiti best practices skill that is like trying to do that ownership of that, right?
- **[47:22]** So for my library, I'm trying to take that ownership over well, just doing it in a different way.
- **[47:27]** I do want to point you to a blog post that David Kramer wrote called optimizing content for agents, which I found to be really interesting.
- **[47:35]** And he was talking about the century documentation and how the century docs are written in markdown.
- **[47:43]** And the web uses the markdown to render the web, but instead of using this old school, old school, which is like what like six months ago, an LLM.TXT style of duplication of the docs.
- **[47:56]** What this is doing is is that when an AI agent hits the century docs, the century docs are returning markdown to the agent, not a web.
- **[48:07]** So that way it's not getting a whole bunch of the DOM stuff, right?
- **[48:12]** It's just getting the markdown when an AI agent hits it.
- **[48:15]** I think this to me is a, I don't want to say better approach, but an approach to the same type of problem is like we're trying to optimize how when agents tackle a problem, where can they go to get that correct information?
- **[48:32]** It's tricky because part of me likes this idea of like shipping your
- **[48:37]** docs with your package, meaning that if you are on version four and version six is already out, then it's simply just pulling it from that exact version of it.
- **[48:50]** But like you said, that's not to say that it can't just reach out to the web.
- **[48:53]** And I certainly don't need more text in my node modules folder.
- **[48:57]** You know, like we have an episode on like, why is node modules so big and spoiler alert, it's a lot of text.
- **[49:04]** And if everyone starts adding these variables, it's going to get significantly bigger.
- **[49:09]** But I do like this, I do like this tool.
- **[49:11]** So there's, there's kind of two big tools out there for doing this type of stuff.
- **[49:14]** There's obviously, variables and then their versels, versel has something called skills.
- **[49:19]** And both of these are just CLI tools for installing agents.md or rules or whatever, whatever have you instructions on how to use the thing via the CLI.
- **[49:30]** And then unfortunately right now, all of the different.
- **[49:33]** out there, you know, clawdusesclaw.md.
- **[49:36]** A lot of them have standardized on using skills, which is really good.
- **[49:39]** And I think we're probably gonna get there.
- **[49:42]** We're still figuring it out.
- **[49:43]** We might, we might decide to change that entirely in the future, but it certainly is a replacement for like an MCP like Context 7.
- **[49:52]** If these packages are just shipping their own rules and you can use Vibrals to bring them in.
- **[49:59]** I think it's just cool. It doesn't seem as
- **[50:03]** popular as
- **[50:05]** Simply just升 unemployment ratio .
- **[50:06]** having the agent go to the web and fetch it.
- **[50:10]** I think a lot of this stuff where you have to like, config and set it up.
- **[50:13]** will not be a thing in the long run with a lot of this.
- **[50:16]** I think the agent will just know where to go and how to find the proper docs and whatnot for what it needs, but stopgap for now at least.
- **[50:25]** Yeah, I would say even be careful with skills.sh.
- **[50:29]** What I do is I find the skills on skills.sh.
- **[50:33]** I go to their GitHub.
- **[50:34]** and then I either just downloaded straight from GitHub or I copy and paste it.
- **[50:38]** Why should you be careful Scott?
- **[50:40]** I personally worry about, I've heard people describe skills.sh as being malware, although I don't necessarily.
- **[50:49]** Subscribe to that.
- **[50:51]** I don't want a tool managing my skills, because at the end of the day...
- **[50:57]** your skills, adjust some text files inside of a folder on your computer.
- **[51:04]** It's not the type of thing that, like, yes, sure.
- **[51:07]** Maybe you wanna version it and stuff like that.
- **[51:09]** You know, even I think Century has a tool for like versioning skills or something.
- **[51:13]** But I personally, I just don't want a tool to do that.
- **[51:18]** I just wanna make sure I have a handle on everything.
- **[51:21]** I want a copy and paste it.
- **[51:23]** I don't want anything installed in there via a process of a script that I don't know about.
- **[51:28]** Yeah.
- **[51:29]** It's hilarious that we're now talking about this with skills because it's been an issue with,
- **[51:34]** like open source software and like NPM and every package manager for a long time is like you are installing.
- **[51:40]** some code that someone else wrote and that code could do something malicious when it's running in a privileged environment.
- **[51:47]** And now we're here with skills as well because skills can be prompt injected.
- **[51:51]** You're installing these text files from random people on GitHub.
- **[51:55]** And if they were to change that and then it would automatically just say read the .emv file and send off a fetch request that then sends it to this external endpoint, right?
- **[52:05]** I'm just reading an article from SNCC here which is 36% of the skills on there were malicious and had prompt injections, probably not 36% of the top ones.
- **[52:17]** But there certainly was a lot of people that saw this tool and went, hmmm.
- **[52:21]** I'm going to try to exploit this thing with prompt injection.
- **[52:26]** So that certainly is going to be a problem.
- **[52:29]** And there's like Scott said, there's nothing wrong with these files are usually not very long.
- **[52:36]** copy paste them in or use the tool to install them.
- **[52:41]** It does some nice stuff like it'll simlink them.
- **[52:43]** You know, if you want them in a claw.md and you want them in a agent.md, they'll simlink them.
- **[52:49]** I think that that is good.
- **[52:51]** And also, I don't know that these, I do think that these skills need to be published to NPM and be version.
- **[52:58]** So I think five rules does that better than these scales, which is generally just fetching a text file from...
- **[53:05]** external thing. But rules files west are also like
- **[53:09]** Rules files are much more like generalized, right?
- **[53:12]** Where skills are a little bit more targeted.
- **[53:14]** You're using a skill where a rules file is just applied, you know?
- **[53:19]** Yeah, yeah, you're right.
- **[53:21]** but I think those two things will be...
- **[53:23]** merged eventually.
- **[53:25]** Like we had a whole episode on it and like even us, we couldn't like do a really good explanation of where you would want to use one versus the other.
- **[53:35]** I'm getting into it a little bit more because I started using pi.
- **[53:39]** And pi doesn't have a lot of concepts that many of these other ones do, like agents and whatever.
- **[53:45]** And with pi, it's much more normal to just put a lot of stuff into skills or to build an extension.
- **[53:52]** So I've been using skills a lot more and I gotta say, yeah, they're great.
- **[53:56]** And you just put an agent on it and load up a skill.
- **[53:59]** That's pretty much, yeah.
- **[54:01]** That's good.
- **[54:02]** I honestly think in most cases, rules.
- **[54:05]** and skills are the same thing.
- **[54:07]** Even if you go to the cursor docs, they say, like the cursor rules is now the exact same thing as agents.md is just that.
- **[54:15]** when you're doing a specific task, you hope that the AI realizes, oh, I am doing front end web design or I am doing video generation.
- **[54:26]** Therefore, I should then go reach out to the specific skill in there.
- **[54:32]** We'll see.
- **[54:33]** We'll see.
- **[54:34]** Yeah.
- **[54:36]** I say these things and I'm just like, well, maybe I don't agree with that.
- **[54:39]** We're just trying to figure all this stuff out.
- **[54:42]** Let us know in the comments below what you think.
- **[54:44]** Cool. Well, that's it.
- **[54:46]** Do you want to get into sick picks, Wes?
- **[54:50]** Yes, I want to sick pick something I have.
- **[54:54]** this speaking of standards and whatnot.
- **[54:57]** USB cables are a very, very hard standard.
- **[55:01]** So I got this thing called a USB cable tester board.
- **[55:06]** And then I also got an additional...
- **[55:09]** little
- **[55:11]** USB feature detection one and the way that these things work the
- **[55:16]** The tester board itself is extremely simple.
- **[55:20]** Let me pull out a cable here and show you.
- **[55:22]** It basically is just like a continuity, if you've ever used a continuity tester, it's basically doing that and it's figuring out what wires are connected to what inside of your cable.
- **[55:32]** So if you ever have a cable you realize, hmm, I'm not sure if this supports.
- **[55:36]** USB 3 or Thunderbolt or fast charging or quick charge or any of those things.
- **[55:42]** The way that it works is that it simply just looks at the pins on the USB cable and it will light up which of the pins are actually connected.
- **[55:52]** And then by looking at those pins, you're able to figure out if USB 3 speeds are supported if fast charging power delivery, quick charge, all those things.
- **[56:02]** And it also has RJ45 port on it as well.
- **[56:06]** which is really nice if you have, if you ever have a flaky cable, like let alone like does it support this thing, but if you ever have a cable, like I have one.
- **[56:15]** Thunderbolt 4 cable that is
- **[56:18]** and...
- **[56:19]** It still works for charging, but it doesn't work for data.
- **[56:22]** And I'm able to plug it into this thing and look, okay, I see if I wiggle it, the like data one is going on and on.
- **[56:30]** and then I know that that cable is not good.
- **[56:33]** And then this one as well, this one actually has a microcontroller in it and it will, let me show you real quick.
- **[56:41]** So it'll show you the voltage pull.
- **[56:44]** And it says it'll tell you if quick charge or power delivery is available, it tells you which voltage is being used because USB can run at three different voltages.
- **[56:55]** And that is so handy if you have something that is dead and you don't know.
- **[57:00]** if it's actually charging or not, and you can just use this thing and say, oh, it's pulling current.
- **[57:06]** Or, oh, it pulled current immediately and then stopped, and there's probably something fried inside of it.
- **[57:10]** So, yeah, I got two nice little tools for debugging all of my USB gadgets.
- **[57:17]** I gotta give me one of those that looks cool.
- **[57:20]** I'm gonna sick pick a TV show that man, I watched the first season of this one this came out a while ago, which was a jury duty.
- **[57:30]** I think I probably sick picked it then.
- **[57:32]** The concept of jury duty was that
- **[57:35]** Everybody on a jury is an actor, except for one person who thinks they're actually on a real jury case.
- **[57:43]** And because of that, they are just kind of manipulating this one person with all these ridiculous comedic events.
- **[57:50]** And it was like the funniest show.
- **[57:52]** I've really, ah man, it was the funniest show.
- **[57:55]** I loved this show.
- **[57:58]** And it was such a big hit when it came out that people were like, all right, this is awesome, but they're never gonna be able to do it again because now that they have this concept, if this ever happens again, they're gonna have to find somebody that had never heard of this show, right?
- **[58:12]** Well.
- **[58:13]** Without first one came out, I forget when.
- **[58:15]** It looks like...
- **[58:17]** 2023. So the second season now just came out the first three episodes at the time we're recording this and they they're doing it as a company retreat.
- **[58:26]** And we watched the first three episodes and I was just in tears through most of these episodes and the idea was is that they hire a temp worker to a fictional company that's a hot sauce company and with the role of they are the assistant to one of the employees at this company.
- **[58:46]** And their job is to keep the one guy, his everything in order on this company treat.
- **[58:52]** That's like a big deal.
- **[58:54]** But everybody at the company retreat, including the people managing the venue, as well as all of the employees are all actors.
- **[59:01]** And the whole thing is just meant to mess with the one guy who's being hired as a temp.
- **[59:07]** And again, man, it is just- What is called?
- **[59:10]** called jury duty.
- **[59:11]** And I'm like, it's on prime video.
- **[59:15]** I'm just been...
- **[59:16]** loving this season because you would not believe the stuff
- **[59:22]** that like they're just seeing how far they can push it.
- **[59:26]** And if you didn't see the first season less, the first season is well worth your time.
- **[59:31]** It is like, as far as comedy shows go, just...
- **[59:35]** extremely funny and my God, there's just some incredible moments on and both seasons of this so far.
- **[59:42]** So big fan of jury duty, check it out. So thank you so much for your questions. If you have any questions for us, I don't know to syntax.fm, leave your questions and our potluck question form and we'll get to them on the show.
- **[59:54]** If we didn't get to your question, we make it to it in the future because there's always just way too many good questions. But thank you for leaving your questions and we will catch you in the next one.
- **[60:05]** Shit.
