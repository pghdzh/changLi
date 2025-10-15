<template>
  <div class="megumi-page" aria-live="polite">
    <main class="container" ref="mainContainer">
      <!-- HERO 区域 -->
      <header class="hero" role="banner">
        <div class="left-col">
          <button class="avatar-card">
            <picture class="avatar-wrap">
              <img src="@/assets/images1/1.webp" class="avatar" loading="lazy" decoding="async" />
            </picture>
            <div class="avatar-meta">
              <span class="avatar-badge">共鸣能力：丹煌离火</span>
            </div>
          </button>
        </div>
        <div class="right-col" ref="metaCard">
          <div class="hero-panel">
            <h1 class="name">
              长离 <span class="jp">（今州令尹参事）</span>
            </h1>
            <p class="subtitle">
              焰落不息，誓言仍燃于心。
            </p>
            <p class="lead">
              终生浴火，总有一日会将己身燃烧殆尽的策士。为达成心中夙愿，她将自己置于棋盘之上，尽可能地算得更远。
            </p>
          </div>
        </div>
      </header>

      <!-- 内容卡片区 -->
      <section id="personality" class="card personality">
        <div class="card-head">
          <h2>核心机制</h2>
        </div>
        <ul class="traits">
          <li>
            【离火】：血量上方上限4点的能量条。长离根据离火点数提供热熔伤害加成。攒满4点【离火】后，重击释放特殊重击【焚身以火】
          </li>
          <li>
            【离火】获取方式：第4段平A 、E技能【心眼·劫】以及变奏入场能为长离提供12秒的【心眼】buff
          </li>
          <li>
            【心眼】：持续期间，长离使用平A派生【心眼·征】，或是跳跃派生【心眼·冲】，命中敌人后获得一点【离火】。

            长离释放共鸣解放【离火照丹心】后，会直接获得4点【离火】。
          </li>
          <li style="font-weight: bold">输出流程：</li>
          <li>基础流程：E \变奏入场-【心眼·冲】-跳跃打断-AAAA-【心眼·冲】-A下落攻击-AA-【心眼·征】-E-【心眼·征】-Z【焚身以火】</li>
          <li>
            小技巧：长离的【心眼·冲】可以用跳跃打断，因此单人连招中使用跳跃缩减时间。
          </li>
          <li>
            技能缩写：普攻 = A，重击 = Z，共鸣技能 = E，共鸣解放 = R，声骸 = Q
          </li>
        </ul>
      </section>

      <section id="overview" class="card overview">
        <div class="card-head">
          <h2>角色背景与形象</h2>
        </div>
        <div class="two-col">
          <div class="text-block">
            <h3>背景</h3>
            <p>
              长离幼时，其所生活的村子被残象潮毁灭，被村民藏匿在防火的地下室的中的她是这场灾难的唯一幸存者。一场雨所带来的机遇与长离的救世之心使长离与玄渺结识，并拜其为师，得师赐名“长离”。在玄渺最后的时间里，他为长离指明了未来的道路，让长离思虑周期，谋定后动，独善其身，待长离羽翼丰满之时为漂泊者指明方向，方可得到长离想要看到的海晏河清。玄渺仙逝以后，得到玄渺指引的长离便开始在摸爬滚打中偷学百家之长，悟百家之道，被书斋老板满街追打也是常事学会了以智避险，一步一筹行于刀尖，孑然一身却也跌跌撞撞的长大，成为了今汐的师傅，教导一州之主，担任前明庭中政省长史，现今州令尹参事。
            </p>
          </div>
          <div class="text-block">
            <h3>外貌／着装</h3>
            <p>
              长离有着一头红色长发，扎着长长的马尾，朱红的发尾多火焰色的发尾如同传说中灵鸟的尾羽在长离身后轻轻摆动，显得慵懒随性，头发两侧戴着金色的三环结发饰，金色的眼睛中有着多重的黑色圆环，与黑色瞳孔。受到共鸣能力的影响，长离面色潮红，全身的肤色较于常人也更为的红润且总是带着温和的笑意。着装上，长离身着一件红白黑三色为主的外袍，袍子下摆被分割为数份。
            </p>
          </div>
        </div>
      </section>

      <section id="resonance-status" class="card overview">
        <div class="card-head">
          <h2>共鸣状况</h2>
        </div>
        <div class="two-col">
          <div class="text-block">
            <p>
              无明确共鸣时间，主诉于幼年时期已出现操控火焰的能力，并多次出现控制不当的情况，成年后频率稳定性大幅提高。对象声痕位于胸部下方，共鸣后部分体征发生明显变化，主诉发梢与发尾褪色、局部出现翎羽状表征，左臂受共鸣能力影响出现朱红色灼痕并伴随火焰色光流。共鸣频谱检验图与火焰接近，测试有较强的谐振反应，无法完全确认共鸣源，暂未发现与该对象相似率超过60%的共鸣频谱检测图。其共鸣能力表征与瑝珑神话传说中的灵鸟类似，推测为对象幼年接触到相关内容影响所致。对象可将火焰凝聚为翎状武器，完全释放能力时催生温度极高、破坏范围极广的火焰，其表征为额前出现单眼雀翎状火焰，主诉称其为“心眼”。受试样本拉贝尔曲线呈稳定上升态，波形检测呈现椭圆形波动，时域表示稳定，最终趋向稳定波动，未见异常波形，最终趋向稳定波动，检测结果判断为自然型共鸣者。诊断结果：超频临界值高，稳定性高，无超频风险。存在过往超频史，最高等级：轻。超频时间主诉为童年期，曾长期处于高度危险的环境中，并多次因自卫或保护其他人员而进入爆发状态，致使对象受能力反噬影响，情况严重时危及生命。超频后遗症表征为左臂烧灼痕迹，频率波动随对象成长过程中对能力的控制力增强而趋于稳定，未有进一步扩散，但能力爆发造成的身体伤害不可逆，或导致对象寿命大幅缩短。建议按时进行身体检查，暂无需心理辅导。
            </p>
          </div>
        </div>
      </section>

      <!-- 新增三大主题区域 -->
      <section id="food" class="card special-section">
        <div class="card-head">
          <h2>炽翎辣肉</h2>
        </div>
        <div class="content two-col">
          <div class="img-area">

            <img src="./img/food.png" alt="炽翎辣肉" loading="lazy" decoding="async" />
          </div>
          <div class="text-block">
            <p>
              长离喜爱的菜肴，比寻常做法加了更多辣椒。 赤红的色泽令辣度肉眼可见，灼辣与鲜香仿佛要化作实体从碗中振翅而出，配一碗米饭与一盅桂花酿，佐以瑝都明庭的长街盛景，酣畅淋漓，实乃人间绝妙。
            </p>
            <p>
              在明庭时，这道佳肴除了长离外鲜少有食客尝试，而在虹镇，这是家家户户都会的家常菜，亲朋满座一堂，觥筹交错间，这道菜便也有了另一番温暖滋味。
            </p>
          </div>
        </div>
      </section>
      <section id="shattered-crystal" class="card special-section">
        <div class="card-head">
          <h2>《乘霄乌鹭集》</h2>
        </div>
        <div class="content two-col">
          <div class="img-area">

            <img src="./img/item1.png" alt="《乘霄乌鹭集》" loading="lazy" decoding="async" />
          </div>
          <div class="text-block">
            <p>一本陈旧的棋谱，记载着一位奇人在乘霄山中留下的传世对局。</p>
            <p>
              其纸页已在漫长岁月中泛黄，边缘也有着被反复翻阅过无数次而产生的残损，它曾经随着主人一起走过无数危险孤寂的荒途，如今已被重新修整装订，增添新纸，记载了一场全新的棋局。
            </p>
            <p>
              如今所寻之人已在面前，未完的棋局也得以继续，只要对弈者仍在，这本棋谱便会继续更新下去。
            </p>
          </div>
        </div>
      </section>

      <section id="cyclic-clock" class="card special-section">
        <div class="card-head">
          <h2>火浣布</h2>
        </div>
        <div class="content two-col">
          <div class="img-area">
            <img src="./img/item2.png" alt="火浣布" loading="lazy" decoding="async" />
          </div>
          <div class="text-block">
            <p>
              材质未知的纺织物，触感柔软，据说能够浴火不焚，经火焰涤荡后的布料反而会更加洁净纯白，皓然凝雪。纺织这种布料所需的植物在悲鸣中覆灭，如今已难以寻得，因此现存的每一匹都珍贵无比。
            </p>
            <p>
              继任大典后，今州的新任令尹将它作为礼物赠予了自己的恩师。
            </p>
          </div>
        </div>
      </section>

      <section id="collective-album" class="card special-section">
        <div class="card-head">
          <h2>心火羽</h2>
        </div>
        <div class="content two-col">
          <div class="img-area">
            <img src="./img/item3.png" alt="心火羽" loading="lazy" decoding="async" />
          </div>
          <div class="text-block">
            <p>长离以心火所化的羽毛，共有一对，如今其中一枚已赠予了今州的贵客，另一枚则一直被长离贴身收藏。</p>
            <p>
              看上去如同在炽热地燃烧，但捧在手心时感受到只是无比熨贴的温暖，代表着长离对其共鸣能力已臻化境的控制力，与她的外袍一样，是完全由火焰织就的造物。
            </p>
            <p>
              很久以前，在小女孩喜欢的枕边故事中，浴火而生的灵鸟会将最为珍贵的尾翎赠予心中重要的存在。
            </p>
          </div>
        </div>
      </section>

      <section id="timeline" class="card timeline" aria-labelledby="timeline-heading">
        <div class="card-head">
          <h2 id="timeline-heading">共鸣链详解</h2>
        </div>
        <div class="tl-wrap" role="list">
          <div class="vl" aria-hidden="true"></div>
          <ol class="tl-list">
            <li v-for="(ev, i) in timeline" :key="i" class="tl-item" role="listitem" tabindex="0" :aria-label="``">
              <div class="tl-card">
                <h3 class="tl-title">第 {{ i + 1 }} 链 · {{ ev.title }}</h3>
                <p class="tl-desc">{{ ev.desc }}</p>
              </div>
            </li>
          </ol>
        </div>
      </section>

      <section class="ll-stories" aria-label="长离的故事合集">
        <div class="card" role="region" aria-labelledby="stories-title">
          <header class="card-head">
            <h2 id="stories-title" class="card-title">长离纪事</h2>


          </header>

          <main class="card-body">
            <transition name="fade-slide" mode="out-in">
              <article :key="items[current].id" :id="`panel-${items[current].id}`" class="story" role="tabpanel"
                :aria-labelledby="`tab-${items[current].id}`">
                <h3 class="story-head">{{ items[current].title }}</h3>
                <div class="story-content" v-html="items[current].content"></div>
              </article>
            </transition>
          </main>

          <footer class="card-foot">
            <button class="nav-btn" @click="prev" aria-label="上一则故事">‹</button>
            <div class="pager">{{ current + 1 }} / {{ items.length }}</div>
            <button class="nav-btn" @click="next" aria-label="下一则故事">›</button>
          </footer>
        </div>
      </section>
    </main>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted } from "vue";
import gsap from "gsap";
import { ScrollTrigger } from "gsap/ScrollTrigger";
gsap.registerPlugin(ScrollTrigger);

const timeline = ref([
  {
    title: "隐我所思",
    desc: `施放共鸣技能赫羽三相或重击焚身以火时，长离的抗打断能力提升，且造成的伤害提升10%。`,
  },
  {
    title: "循我所望",
    desc: `获得【离火】时，长离的暴击提升25%，持续8秒。`,
  },
  {
    title: "据我所闻",
    desc: `共鸣解放离火照丹心造成的伤害提升80%。`,
  },
  {
    title: "饰我所言",
    desc: `施放变奏技能后，队伍中的角色攻击提升20%，持续30秒。`,
  },
  {
    title: "舍我所得",
    desc: `重击焚身以火倍率提升50%，造成的伤害提升50%。`,
  },
  {
    title: "成我所谋",
    desc: `共鸣技能赫羽三相、重击焚身以火和共鸣解放离火照丹心攻击造成伤害时额外忽视目标40%防御。`,
  },
]);

const reducedMotion =
  window.matchMedia &&
  window.matchMedia("(prefers-reduced-motion: reduce)").matches;


const items = [
  {
    id: "legend",
    title: "传言之外",
    content: `
      <p>传言之外近两年来到边庭的人总说，很少见到那个传说中的参事长离。</p>
      <p>虽说这位参事大人鲜少在公开场合露面，但作为令尹最信任的佐臣，她的名号在今州无人不知，而关于她的传闻，却千奇百怪，各有说法。</p>
      <p>据说，长离大人在明庭时奏请修葺了各地的悲田院，令那些在悲鸣带来的灾难中失所的流民有了更好的归处；她还进谏中央庭减轻今州的商税，拨发粮款，助今州从弯刀之役中的阴影中恢复民生。</p>
      <p>据说，长离大人刚到边庭时就不动声色地解决了两个盘踞在今州周边已久的流寇势力，仅是一场交谈，一个“显而易见的陷阱”便令两方互相猜忌缠斗，两败俱伤。</p>
      <p>据说，长离大人并非岁主选定令尹之后才任命参事，而是一早便自请下今州代理事务，教导尚年幼的今汐，协助她接过对一个孩子来说过于沉重的权柄。</p>
      <p>“真的假的？我怎么听说，前面那些都是上一任长史的举措，她是因为得罪了大人物，才被贬谪到边关来？”</p>
      <p>“听说她是玄渺真人的亲传弟子，又得上一任长史举荐，游说四州，无人不拜服，怎么可能是个佞臣呢。”</p>
      <p>“什么呀，我听说她是某个隐士大家的后裔，这次来今州，是因为肩负着家族传承的重任……”</p>
      <p>诸如此类讨论甚嚣尘上，长离却似无所觉，偶尔撞见有人议论，她也只是毫不避讳地望过去，目光温和，却仿佛能看穿所有人的想法，反倒让那些人噤了声。</p>
      <p>今州令尹由角选定，竟是个看上去尚未及笄的小姑娘，于是流言便更加猖獗，有人认为她手眼通天，盯上了令尹的地位，来此的目的就是为了控制那个年幼的孩子，取而代之。</p>
      <p>而继任大典之后，长离便将理政之权完全交由今汐，被今汐任命为令尹参事。三年时间，今州政通人和，民众无不感念令尹勤政为民，这之中有多少是参事从旁裨辅，也从未有人知晓。</p>
      <p>见到过长离的人都说，她总是带着温和的微笑，待人接物上至将领下至走卒，态度皆一视同仁，但独处时总是有种不怒自威的气势，令人有些不敢靠近。</p>
      <p>“上次在巡宁所遇见长离大人，发现她一直饶有兴趣地盯着院中的树看，我原本以为她有什么要事在身，却没想到她只是在看蚂蚁搬家……看了整整一个下午……”</p>
      <p>“然后呢？”</p>
      <p>“后来有个被通缉很久的嫌犯头目突然过来自首，吓了我们一跳，起初以为又是什么陷阱，结果那人看到长离大人就像是受到了什么巨大的惊吓，一下子把什么都招了，而且还牵连出许多陈年疑案，这是我入行这么久，接过的最大的一个案子……”</p>
      <p>“那长离大人呢？”</p>
      <p>“不知什么时候离开了，唔，真是一位让人捉摸不透的大人啊……”</p>
    `
  },
  {
    id: "teacher",
    title: "师与臣",
    content: `
      <p>在长离成为今汐的老师之前，曾与年幼的汐有过一面之缘。</p>
      <p>那时长离前往今州，在边庭中行走时偶然听到侧殿中传来隐约的诵读声，有一个年幼的女孩正坐在比她还高的书堆里，努力地记忆那些对她的年纪来说有些艰深的理论。</p>
      <p>“心不知治乱之源者，不可令制法……但要怎样才能做到有独闻之耳，独见之明呢。”</p>
      <p>女孩懵懵懂懂地念出书中的内容，却有些不得其意。那个小小的背影令长离想起了曾经的自己，也是这样努力地汲取着尽可能多的知识，着急地想要长大。她不由得俯下身，为女孩解答她的疑问。</p>
      <p>“汐受益匪浅。” 女孩用稚嫩的嗓音一字一句地道谢，犹豫半响，又抬起头问她，“……那么，要怎样才能成为独当一面的令尹？”</p>
      <p>“为何这样问？”</p>
      <p>“今州人民苦于战事……我想看到，清平盛世。”</p>
      <p>“……这并非易事。”</p>
      <p>“我明白，但我想尽己所能。”</p>
      <p>——我想看到，盛世太平。</p>
      <p>长离看着女孩澄澈的双眼，似乎听到了星火燃烧的声响，微小却坚定。</p>
      <p>“那么，我会助你达成所愿。”</p>
      <p>曾经的小女孩被今州的岁主选定为新的令尹，边庭彻夜亮灯，长离始终陪在今汐身边，作为老师解答疑问，作为参事进言献策。</p>
      <p>对于普通人来说，英雄的故事宏大而遥远，餐桌上的热饭热菜与能够安眠的夜晚才是日子变好的象征。</p>
      <p>“汐，为政不可只高居庙堂，囿于案牍之间，要走到民众中去，聆听他们的心声。”</p>
      <p>“汐明白。”</p>
      <p>于是一件件柴米油盐中的愿望，寄托于今州许愿的习俗，汇聚为一个州城向好转变的开始。长离看着今汐被热情的民众簇拥，萧索之态从他们的脸上尽数褪去，取而代之的是真心实意的幸福与感激。</p>
      <p>今州已得明主，不论是民众还是中央庭，想必都不会再质疑今汐的能力。</p>
      <p>“想到什么，便去做吧。”</p>
      <p>“还有我在。”</p>
      <p>老师，是播种之人，燃烛引路，令星火之光照亮天际。她知道，这个眉宇间仍显稚嫩的女孩，会成长为足以庇护今州全境的，独当一面的令尹。而她，会接过传承的星火，为自己的学生铺平前行的道路。</p>
    `
  },
  {
    id: "alley_full",
    title: "巷中孤羽",
    content: `
<p>延庆斋的常庆老板最近遇到了两件奇事。</p>

<p>一是书店的生意。</p>

<p>瑝珑地大物博，城中的时兴之物总在变化，近些年来人们不再推崇纸质书籍，书店的生意每况愈下。而这几日生意居然奇迹般地回暖，前来店中购书的人也比往常多上许多。</p>

<p>看装束，顾客多为中央庭官员——常庆心中疑惑，便去问店中顾客，得到的答复大多是：“这是长史大人推荐的地方”。</p>

<p>长史？他隐约听说最近中央庭新上任了长史，可他一个小小的书店老板，如何与这高居庙堂的大人有什么联系？</p>

<p>二是书店的门面。</p>

<p>生意萧条下，延庆斋的门面也许久未曾修葺，就连隔壁杂货铺的老板看了都直摇头。</p>

<p>“老常啊，你这门面确实该修整修整了，再破下去，别人真要把你这当烂仓库了！”</p>

<p>“你才烂仓库，虽说规模比不上明庭的书店，但论珍善孤本，我这延庆斋可是最全的！酒香不怕巷子深懂不懂。”</p>

<p>话虽这么说，但常庆心里也犯起了嘀咕，可他一把老骨头干不了重活，干脆趁最近生意回暖，雇个人修理也罢。</p>

<p>可第二天到店时他却惊得愣在了原地，原本陈旧落灰的门面一夜之间不复存在，牌匾与正门焕然一新，描金的“延庆斋”三字在阳光下闪闪发亮。</p>

<p>柜台前放着一个信笺，上面写着感念老板恩情，擅自修缮，聊表谢意。</p>

<p>信笺没有署名，但纸页上朱红色的火羽纹饰却有些眼熟，常庆攥着信笺缓缓坐了下来，陷入了回忆。</p>

<p>那还是在十几年前书店生意兴隆的时候，来来往往的顾客中总有个八九岁的小女娃悄悄溜进店里，捧着书一看就是一下午。</p>

<p>常庆起初不甚在意，开书店嘛，遇上只看不买的客人也是常事，只要不毁书不影响生意就成。</p>

<p>然而连着几日，销量断崖下跌，连进店的客人都少了许多。</p>

<p>“太热了太热了，待不下去，走走走，去别的店看看。”</p>

<p>时值早秋，屋内却热得异常，常庆擦擦额间的汗，仔细找了一圈，才发现书架的阴影中蜷缩着一个瘦小的影子，正捧着书看得聚精会神。</p>

<p>她的面前竟漂浮着一团火，房间里的温度就随着她看书入迷时的神情变换，逐渐走高——常庆看到眼前一幕，顿觉急火攻心。</p>

<p>“你是哪来的熊孩子，想烧我的店？！”</p>

<p>火焰倏然熄灭，女孩被吼声吓得一个激灵，常庆眼睁睁地瞅着她紧攥的那片书页逐渐焦黄，像是要烧起来了——</p>

<p>“你你你……！”</p>

<p>女孩慌忙丢下手中的书，看着纸页上的焦痕，她幼小的脸庞上流露出慌乱而愧疚的神色。</p>

<p>“对，对不起……这里太暗了，我只是想照明……”</p>

<p>眼见自己珍藏的孤本被糟蹋，脑中又划过这几日持续走低的账目，常庆黑着脸抄起书架边的扫帚，作势要打。</p>

<p>想不到这娃娃看着瘦小却窜得比猴还快，常庆追出门去却只看到那火焰色的发尾在空中划过，小小的身影瞬间就挤进了人头攒动的集市中，留下他站在原地气得吹胡子瞪眼睛。</p>

<p>“别让我再看见你！”</p>

<p>“怎么了，火气这么大？”旁边杂货铺的老板笑呵呵地走出来，顺着他的目光看了过去，“哟，那不是那个会控火的小娃娃吗。”</p>

<p>“你认识？她刚刚差点把我店给烧了！”</p>

<p>“哪有那么夸张，那是个好孩子啊，前几天还帮我干了不少活，只为了要些馒头……唉，也是可怜，小小年纪孤身一人，单靠一双腿走到这里，听说她来的地方被残象潮袭击，一个人都没活下来啊，也不知道她是怎么撑下来的……”</p>

<p>“……你咋不早说？”</p>

<p>“你也没问啊！不过看这样子……她应该是不敢再回来了吧。”</p>

<p>小女孩的瘦弱的身影已经消失在了巷尾，坊间传来邻家孩童嬉戏笑闹的声音，常庆看着落在地上的书，心中有些不是滋味。</p>
`
  },

  {
    id: "heyi_full",
    title: "赫奕归林",
    content: `
<p>延庆斋的生意又回到了往常的样子，书客依旧络绎不绝，再也没见过那个缩在书架间默默读书的小女孩。</p>

<p>但让常庆没有想到的是，一周之后，那个孩子居然又回来了。</p>

<p>“这是我所有的钱……不是偷的！是我赚的，不知道够不够赔……抱歉，弄坏了您的书……我想看的书，只有先生这里有，所以才……”</p> 

<p>女孩咬着嘴唇，怯生生地将手中零零碎碎的贝币放在了柜台上。那瘦小的身板还没有柜台高，只能露出半个脑袋，女孩踮着脚的身体微微发抖，而一双明亮得有些倔强的眼睛仍然毫不逃避地注视着他。</p>

<p>“……罢了。”</p>

<p>已经过去这么些天，气也早都消了，常庆摆了摆手，“走吧走吧，钱也拿走，别影响我做生意。”</p>

<p>女孩没有再说话，鞠了一躬后一步三回头地离开，看着女孩瘦弱的背影和破旧的衣衫，常庆沉默片刻，又重重叹了口气。</p>

<p>他起身拎着一盏灯走了过去，和店铺的备用钥匙一起塞到了女孩的手里。</p>

<p>“拿好，也别再用你那火照明了，晚上没客人的时候光明正大地来看。”</p>

<p>“……”</p>

<p>”没让你白看，早上顺便帮我把店给开了。”</p>

<p>“谢，谢谢您……”</p>

<p>那是他第一次见到那个女孩绽放出纯粹的，属于一个孩子的笑容。</p>

<p>那之后，女孩便时常来到延庆斋，她依照约定只在傍晚出现，一来便默默缩在角落读书，到清晨常庆来进店时，总是能看到店中被打扫的一尘不染，井井有条。</p>

<p>到了冬日，女孩便也能在白天来读书了。窗外寒风萧瑟，室内却温暖如春，常庆偶尔会撇一眼女孩在看什么，发现尽是一些说文识字、图解史记之类的内容。</p>

<p>那不甚符合年龄的沉静面庞，也只有在沉浸于书中时才有神采飞扬的时刻。</p>

<p>——后来的某天，女孩不知去向，有人说她遭了残象袭击，有人说她伤重不治，死在了前往明庭的路上。</p>

<p>信笺上的火羽纹样在常庆眼中，愈发像是女孩曾经点起的那团火焰。</p>

<p>如果她还活着……</p>

<p>“老先生？”</p>

<p>常庆从沉思中回神，便看到柜台前站着一个女子，她身长玉立，眸光温和，火焰色的发尾如同传说中灵鸟的尾羽，正随晨风悠悠飘扬。</p>

<p>她将手中的贝币放在柜台，含笑开口。</p>

<p>“我想找的书，只有先生这里有，不知可否借读一日？”</p>

<p>——她不再像那个瘦弱的小女孩了，身姿气度倒是与街头巷尾议论纷纷的，那位官拜长史的年轻人如出一辙。</p>

<p>日头升上屋檐，带来一室晨光，年迈的书店老板笑了起来，摆了摆手，又靠回躺椅中。</p>

<p>“看吧，看吧……就算不给钱，我也赶不动你咯。”</p>
`
  },

  {
    id: "tome_full",
    title: "典籍中的背影",
    content: `
<p>师父似乎一直在追随一个人。</p>

<p>年幼的长离跟着年迈的师父在瑝珑中游历，德高望重的老者本已不打算再授徒讲学，但既已收下了这小徒弟，便要尽心在最后的时间里将全部学识倾囊相授。老人喜爱弈棋，在赶路疲乏的时候，师徒便席地而坐，摆出棋盘对弈。师父大多时间与长离下指导棋，在棋中教导她如何筹策万类，度权量能。</p>

<p>到这时，他总会提起那个与他对弈的旧友，长离明白那人便是师父一直在追随的人，师父收她为徒，也是为了寻找这个人。但若是闻及那人姓甚名谁、现在何处，师父却总是摇摇头。</p>

<p>“时机未到。”</p>

<p>在预感到自己大限将至的那一日，老人将随身携带的棋谱与手记交给了长离，也将遗志传给了自己的徒儿。</p>

<p>“一切因果，皆在棋中。”</p>

<p>于是徒弟接过了师父的遗志，继续前行。</p>

<p>“……斯天人降世，乾坤指掌，纳骸于身。”</p>

<p>“……见金瞳异人于云霞明灭之间。”</p>

<p>“……与角并肩而立，亲密无间。”</p>

<p>她顺着师父留下的手记，一点点向前追寻，发现许多地方都存在着那个人的影子，有时是说书人口中精彩纷呈的故事，有时是史书典籍中只言片语的记载——那个存在如同坚定的基石与灯塔，在数度风雨飘摇的文明中将人们引向更好的未来。</p>

<p>那个人在世界上留下的痕迹，如同一缕难以察觉却坚韧绵长的丝线，将比普通人的寿命漫长得多的岁月与历史一一串联。</p>

<p>“那个特殊之人的出现，势必伴随着足以令你注意到的征兆。那个人，是真正有能力改变这个世界的人。”</p>

<p>而若想达成盛世太平的夙愿，却不能仅靠遥远的幻影，她必须站在足够高足够远的地方，才能有足够强的力量亲手触碰她追寻的未来，完成一路走来所有人托付于她的祈盼与约定，于是她游学四国，雪案萤窗，即使路途遥远艰险，也未曾放弃分毫。</p>

<p>官拜长史之后，她终于获得权限，如愿踏入瑝览类书，在浩如烟海的记载中，她找到了最为关键的信息。</p>

<p>信息所指向的，正是瑝珑的边陲重地，今州。</p>

<p>一切都如同岁主预言那般，贵客造访今州，破局止战，护得今州安宁——曾经只在典籍与传说中出现的模糊背影，如今终于有了确切的形态。</p>

<p>正式相见的那天，她从明庭匆匆赶回今州，见那位贵客正站在渡口，似乎在雨中踌躇。</p>

<p>——只要那位贵客按照角所言前行，我们总会相见。</p>

<p>如今，时机已到。</p>

<p>于是她隐去眼中如潮的情绪，一步一步走向那个人身边。</p>

<p>绘着灵鸟纹样的纸伞轻斜，二人四目相对，天地间只余细雨敲击伞面的声响。</p>

<p>她露出微笑，轻声开口。</p>

<p>“——你是被雨困在了这里吗？”</p>
`
  }
];
const current = ref(0);

function select(index) {
  current.value = index;
  // 将焦点移回当前 tab（便于无障碍与键盘导航）
  const id = `tab-${items[index].id}`;
  const el = document.getElementById(id);
  if (el) el.focus();
}

function prev() {
  current.value = (current.value - 1 + items.length) % items.length;
  select(current.value);
}

function next() {
  current.value = (current.value + 1) % items.length;
  select(current.value);
}



onMounted(() => {
  if (!reducedMotion) {
    const tl = gsap.timeline({ defaults: { ease: "power3.out" } });
    tl.from(".avatar-card", {
      duration: 0.8,
      y: 18,
      opacity: 0,
      scale: 0.99,
    }).from(
      ".right-col > .hero-panel .name",
      { duration: 0.9, x: 24, opacity: 0 },
      "-=0.45"
    );

    gsap.utils.toArray(".tl-item").forEach((it: any, idx: number) => {
      gsap.from(it.querySelector(".tl-card"), {
        scrollTrigger: {
          trigger: it,
          start: "top 85%",
          toggleActions: "play none none reverse",
        },
        opacity: 0,
        y: 22,
        duration: 0.7,
        delay: idx * 0.06,
      });
    });

    gsap.to(".avatar-card", {
      scale: 1.007,
      duration: 2.8,
      yoyo: true,
      repeat: -1,
      ease: "sine.inOut",
    });
  } else {
    document.documentElement.classList.add("reduced-motion");
  }

});

</script>

<style scoped lang="scss">
.megumi-page {
  position: relative;
  min-height: 100vh;
  background: linear-gradient(135deg, #ca633e 0%, #1b0f0c 45%, #a14b2b 100%);
  font-family: "Noto Sans SC", system-ui, -apple-system, "Segoe UI", Roboto,
    Arial;
  color: #eef2f7;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  padding: clamp(20px, 4vw, 80px);
  box-sizing: border-box;
  padding-top: 60px;

  &.reduced-motion * {
    transition: none !important;
    animation: none !important;
  }

  .container {
    position: relative;
    z-index: 10;
    max-width: 1100px;
    margin: 0 auto;
    padding: clamp(12px, 2vw, 40px);
  }
}

/* ========== HERO 区域 ========== */
.hero {
  position: relative;
  display: grid;
  grid-template-columns: 280px 1fr;
  gap: 28px;
  align-items: start;
  padding: 12px;
  margin-bottom: 12px;

  .left-col {
    display: flex;
    flex-direction: column;
    align-items: center;
  }

  .avatar-card {
    background: rgba(255, 255, 255, 0.1);
    border-radius: 16px;
    padding: 12px;
    display: grid;
    justify-items: center;
    gap: 8px;
    border: 1px solid rgba(255, 255, 255, 0.15);
    box-shadow: 0 15px 40px rgba(0, 0, 0, 0.25);
    cursor: pointer;
    transition: transform 0.28s ease, box-shadow 0.28s ease;
    width: 100%;

    &:hover {
      transform: translateY(-6px);
      box-shadow: 0 22px 60px rgba(0, 0, 0, 0.35);
    }
  }

  .avatar-wrap {
    width: 100%;
    display: block;
  }

  .avatar {
    width: 100%;
    height: auto;
    border-radius: 12px;
    display: block;
    box-shadow: 0 10px 30px rgba(0, 0, 0, 0.25);
    transition: transform 0.5s ease;
  }

  .avatar-card:hover .avatar {
    transform: translateY(-4px) scale(1.01);
  }

  .avatar-meta {
    display: flex;
    gap: 8px;
    align-items: center;
    justify-content: center;
    flex-wrap: wrap;
  }

  .avatar-badge {
    font-size: 0.9rem;
    color: #f0e9e1;
    background: rgba(0, 0, 0, 0.25);
    padding: 6px 10px;
    border-radius: 999px;
    border: 1px solid rgba(255, 255, 255, 0.2);
    font-weight: 600;
  }

  .right-col {
    display: flex;
    flex-direction: column;
    gap: 12px;
  }

  .hero-panel {
    position: relative;
    z-index: 9;
    padding: 20px;
    border-radius: 14px;
    background: rgba(0, 0, 0, 0.16);
    backdrop-filter: blur(6px);
    border: 1px solid rgba(255, 255, 255, 0.2);
    box-shadow: 0 15px 45px rgba(0, 0, 0, 0.3);
  }

  .name {
    font-size: clamp(1.8rem, 3vw, 2.8rem);
    margin: 0;
    background: linear-gradient(90deg, #ff6b35 0%, #ff9a66 50%);
    -webkit-background-clip: text;
    background-clip: text;
    color: transparent;
    -webkit-text-fill-color: transparent;
    line-height: 1;
    font-weight: 900;
    letter-spacing: 1px;
    display: inline-block;
    -webkit-text-stroke: 0.3px rgba(0, 0, 0, 0.2);
  }

  .jp {
    font-size: 0.95rem;
    color: #ff9a66;
    margin-left: 8px;
    font-weight: 400;
  }

  .subtitle {
    margin: 6px 0 0;
    font-size: 1.1rem;
    color: #f7712e;
    font-weight: 600;
    line-height: 1.4;
  }

  .lead {
    margin: 10px 0 0;
    color: #ffffff;
    line-height: 1.7;
    max-width: 72ch;
  }
}

/* ========== 通用卡片 ========== */
.card {
  margin-top: 24px;
  background: rgba(0, 0, 0, 0.1);
  border-radius: 14px;
  padding: 24px;
  box-shadow: 0 15px 45px rgba(0, 0, 0, 0.3);
  backdrop-filter: blur(4px);
  color: #ffffff;
  border: 1px solid rgba(255, 255, 255, 0.15);

  &:hover {
    transform: translateY(-8px) scale(1.01);
    box-shadow: 0 25px 70px rgba(0, 0, 0, 0.45);
    transition: transform 0.28s ease;
  }

  h2 {
    margin: 0 0 12px 0;
    font-size: 1.3rem;
    color: #f7c1a7;
  }

  .card-head {
    display: flex;
    justify-content: space-between;
    align-items: center;
    gap: 12px;
  }
}

/* personality 列表样式 */
.personality .traits li {
  padding: 12px 0;
  border-bottom: 1px dashed rgba(255, 255, 255, 0.25);
  font-size: 1rem;
  color: #dce4e8;
}

/* overview 双栏布局 */
.overview .two-col {
  display: flex;
  gap: 24px;
  flex-wrap: wrap;
  margin-top: 12px;
}

.overview .text-block {
  flex: 1 1 40%;
  min-width: 240px;
  line-height: 1.6;
}

/* special-section：三大主题区公共布局 */
.special-section {
  position: relative;
  overflow: hidden;

  .content {
    display: flex;
    gap: 24px;
    align-items: center;
    margin-top: 16px;
  }

  .img-area {
    flex: 0 0 200px;

    img {
      width: 100%;
      height: auto;
      border-radius: 12px;
      box-shadow: 0 12px 36px rgba(0, 0, 0, 0.3);
    }
  }

  .text-block {
    flex: 1;
    line-height: 1.6;
  }
}



.ll-stories {


  .card {
    width: min(1200px, 100%);

    border-radius: 12px;
    box-shadow: 0 8px 30px rgba(6, 6, 6, 0.5), inset 0 1px 0 rgba(255, 160, 110, 0.02);
    color: #f5e8dc;
    overflow: hidden;
    display: flex;
    flex-direction: column;
  }

  .card-head {
    padding: 14px 16px;
    border-bottom: 1px solid rgba(255, 120, 70, 0.04);
    display: flex;
    flex-direction: column;
    gap: 12px;

    .card-title {
      font-size: 24px;
      font-weight: 700;
      color: #ffb890;
      letter-spacing: 0.6px;
    }

  }

  .card-body {
    padding: 18px 16px;
    min-height: 160px;
    position: relative;
    background: linear-gradient(180deg, rgba(0, 0, 0, 0.00), rgba(8, 4, 3, 0.02));
  }

  .story {
    color: rgba(245, 232, 220, 0.95);

    .story-head {
      font-size: 18px;
      color: #ff9a66;
      font-weight: 700;
      margin-bottom: 8px;
    }

    .story-content {
      line-height: 1.7;
      color: rgba(245, 232, 220, 0.92);
    }
  }

  .card-foot {
    display: flex;
    align-items: center;
    justify-content: space-between;
    gap: 8px;
    padding: 10px 14px;
    border-top: 1px solid rgba(255, 120, 70, 0.03);
    background: linear-gradient(180deg, rgba(0, 0, 0, 0.02), rgba(0, 0, 0, 0.04));

    .nav-btn {
      background: transparent;
      border: 1px solid rgba(255, 120, 70, 0.06);
      color: #ffb890;
      padding: 6px 10px;
      border-radius: 6px;
      cursor: pointer;
    }

    .pager {
      color: rgba(245, 232, 220, 0.7);
      font-size: 13px;
      font-weight: 600;
    }
  }

  /* 切换动画：淡入并轻微上移 */
  .fade-slide-enter-active,
  .fade-slide-leave-active {
    transition: all 320ms cubic-bezier(.2, .9, .2, 1);
  }

  .fade-slide-enter-from {
    opacity: 0;
    transform: translateY(6px);
  }

  .fade-slide-enter-to {
    opacity: 1;
    transform: translateY(0);
  }

  .fade-slide-leave-from {
    opacity: 1;
    transform: translateY(0);
  }

  .fade-slide-leave-to {
    opacity: 0;
    transform: translateY(-6px);
  }
}



/* timeline 样式（略微调整色调） */
.timeline {
  position: relative;

  .tl-wrap {
    position: relative;
    padding: 24px 12px 12px 96px;
  }

  .vl {
    position: absolute;
    left: 80px;
    top: 32px;
    bottom: 20px;
    width: 4px;
    border-radius: 6px;
    background: linear-gradient(180deg,
        rgba(255, 255, 255, 0.2),
        rgba(255, 255, 255, 0.1));
    box-shadow: 0 6px 18px rgba(255, 255, 255, 0.1),
      inset 0 0 10px rgba(255, 255, 255, 0.05);
    opacity: 0.9;
  }

  .tl-list {
    list-style: none;
    margin: 0;
    padding: 0;
  }

  .tl-item {
    position: relative;
    display: flex;
    gap: 18px;
    align-items: flex-start;
    margin: 20px 0;
    transition: transform 0.28s ease, box-shadow 0.28s ease;
  }

  .tl-card {
    background: rgba(255, 255, 255, 0.05);
    border-radius: 12px;
    padding: 16px 20px;
    box-shadow: 0 10px 30px rgba(0, 0, 0, 0.25);
    width: calc(100% - 130px);
    border-left: 4px solid transparent;
    transition: transform 0.28s ease, box-shadow 0.28s ease,
      border-color 0.28s ease;
    position: relative;
    z-index: 2;
  }

  .tl-item:hover .tl-card,
  .tl-item:focus-within .tl-card {
    transform: translateY(-8px);
    box-shadow: 0 20px 60px rgba(0, 0, 0, 0.35);
    border-left-color: rgba(255, 255, 255, 0.3);
  }

  .tl-title {
    margin: 0 0 6px 0;
    font-size: 1.1rem;
    color: #ca633e;
    display: flex;
    align-items: baseline;
    gap: 8px;
  }

  .tl-desc {
    margin: 0;
    color: #fae4d5;
    white-space: pre-wrap;
  }
}

/* 小屏响应式调整 */
@media (max-width: 980px) {
  .hero {
    grid-template-columns: 1fr;
    gap: 18px;
    padding: 8px;
  }

  .avatar-card {
    width: 140px;
    padding: 8px;
  }

  .hero-panel {
    padding: 14px;
  }

  .name {
    font-size: 1.8rem;
  }

  .subtitle {
    font-size: 1rem;
  }

  .lead {
    font-size: 0.95rem;
  }

  .card {
    padding: 18px;
    border-radius: 12px;
  }

  .overview .two-col {
    flex-direction: column;
  }

  .special-section .content {
    flex-direction: column;
  }

  .ll-stories {

    .card {
      border-radius: 10px;
    }

    .card-head {
      padding: 12px 12px;
    }

    .card-body {
      padding: 14px 12px;
      min-height: 180px;
    }

    .card-foot {
      padding: 8px 12px;
    }
  }

  .timeline {
    /* 取消可能的 overflow 隐藏 */
    overflow: visible;
  }

  .timeline .tl-wrap {
    padding-left: 14px !important;
    padding-right: 14px;
    /* 也给右边留出 padding */
    /* 如果 tl-wrap 是 flex 子项，强制可压缩 */
    min-height: 0;
  }

  .timeline .vl {
    display: none !important;
  }

  .timeline .tl-list {
    /* 保证列表不被挤出 */
    overflow: visible;
  }

  .timeline .tl-item {
    display: block !important;
    margin: 14px 0 !important;
    width: 100%;
  }

  .timeline .tl-card {
    width: 100% !important;
    padding: 14px !important;
    border-left: 0 !important;
    box-shadow: 0 10px 30px rgba(0, 0, 0, 0.25);
    transform: none !important;
  }
}
</style>
