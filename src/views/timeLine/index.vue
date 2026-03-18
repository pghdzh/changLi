<template>
  <div class="changli-page" aria-live="polite">
    <!-- 背景火焰粒子层 -->
    <div class="bg-particles">
      <span
        v-for="i in 24"
        :key="i"
        class="particle"
        :style="particleStyle(i)"
      ></span>
    </div>

    <!-- 底部火焰光带 -->
    <div class="page-glow"></div>

    <!-- 棋盘背景装饰 -->
    <div class="board-bg"></div>

    <!-- 翎羽装饰 -->
    <div class="feather-decoration left"></div>
    <div class="feather-decoration right"></div>

    <main class="container" ref="mainContainer">
      <!-- HERO 区域（增强版） -->
      <header class="hero" role="banner">
        <div class="left-col">
          <div class="avatar-card">
            <div class="avatar-flame"></div>
            <picture class="avatar-wrap">
              <img
                src="@/assets/images1/slty1 (8).webp"
                class="avatar"
                loading="lazy"
                decoding="async"
              />
            </picture>
            <div class="avatar-meta">
              <span class="avatar-badge">共鸣能力：丹煌离火</span>
              <span class="avatar-badge">今州令尹参事</span>
            </div>
          </div>
        </div>
        <div class="right-col" ref="metaCard">
          <div class="hero-panel">
            <h1 class="name">长离 <span class="jp">（今州令尹参事）</span></h1>
            <p class="subtitle">“焰落不息，誓言仍燃于心。”</p>
            <p class="lead">
              终生浴火，总有一日会将己身燃烧殆尽的策士。为达成心中夙愿，她将自己置于棋盘之上，尽可能地算得更远。
            </p>
            <!-- 火焰装饰线 -->
            <div class="hero-flame-line"></div>
          </div>
        </div>
      </header>

      <!-- 核心机制卡片（优化样式） -->
      <section id="personality" class="card personality">
        <div class="card-head">
          <h2>核心机制</h2>
          <span class="flame-icon"></span>
        </div>
        <ul class="traits">
          <li
            v-for="(item, index) in mechanismItems"
            :key="index"
            :style="{ '--item-index': index }"
          >
            <span class="flame-bullet"></span>
            {{ item }}
          </li>
        </ul>
      </section>

      <!-- ========== 板块一：角色背景与形象 · 古籍书卷风格 ========== -->
      <section id="overview" class="card overview">
        <div class="card-head">
          <div class="title-group">
            <!-- 古籍标题装饰 -->
            <span class="title-ornament left"></span>
            <h2>角色志·卷首</h2>
            <span class="title-ornament right"></span>
          </div>
          <span class="flame-icon"></span>
        </div>

        <div class="two-col">
          <!-- 左侧：背景（线装书样式） -->
          <div class="text-block background-block">
            <div class="block-header">
              <span class="block-number">壹</span>
              <h3>生平纪事</h3>
              <span class="header-line"></span>
            </div>
            <div class="scroll-container">
              <p class="classical-text">
                <span class="dropcap">长</span
                >离幼时，其所生活的村子被残象潮毁灭，被村民藏匿在防火的地下室的中的她是这场灾难的唯一幸存者。一场雨所带来的机遇与长离的救世之心使长离与玄渺结识，并拜其为师，得师赐名“长离”。在玄渺最后的时间里，他为长离指明了未来的道路，让长离思虑周期，谋定后动，独善其身，待长离羽翼丰满之时为漂泊者指明方向，方可得到长离想要看到的海晏河清。玄渺仙逝以后，得到玄渺指引的长离便开始在摸爬滚打中偷学百家之长，悟百家之道，被书斋老板满街追打也是常事学会了以智避险，一步一筹行于刀尖，孑然一身却也跌跌撞撞的长大，成为了今汐的师傅，教导一州之主，担任前明庭中政省长史，现今州令尹参事。
              </p>
            </div>
            <!-- 书页装订线 -->
            <div class="binding-line"></div>
            <!-- 古籍印章 -->
            <div class="seal-mark">玄渺真传</div>
          </div>

          <!-- 右侧：外貌着装（绢本质感） -->
          <div class="text-block appearance-block">
            <div class="block-header">
              <span class="block-number">贰</span>
              <h3>仪容考</h3>
              <span class="header-line"></span>
            </div>
            <div class="portrait-frame">
              <!-- 工笔画风格装饰线 -->
              <div class="frame-corner top-left"></div>
              <div class="frame-corner top-right"></div>
              <div class="frame-corner bottom-left"></div>
              <div class="frame-corner bottom-right"></div>
              <p class="elegant-text">
                长离有着一头红色长发，扎着长长的马尾，朱红的发尾多火焰色的发尾如同传说中灵鸟的尾羽在长离身后轻轻摆动，显得慵懒随性，头发两侧戴着金色的三环结发饰，金色的眼睛中有着多重的黑色圆环，与黑色瞳孔。受到共鸣能力的影响，长离面色潮红，全身的肤色较于常人也更为的红润且总是带着温和的笑意。着装上，长离身着一件红白黑三色为主的外袍，袍子下摆被分割为数份。
              </p>
            </div>
            <!-- 羽毛笔装饰 -->
            <div class="quill-decoration"></div>
          </div>
        </div>

        <!-- 底部卷轴装饰 -->
        <div class="scroll-decoration">
          <span class="scroll-left"></span>
          <span class="scroll-text">· 今州令尹参事 ·</span>
          <span class="scroll-right"></span>
        </div>
      </section>

      <!-- ========== 板块二：共鸣状况 · 档案鉴定风格（精简版） ========== -->
      <section id="resonance-status" class="card resonance">
        <div class="card-head">
          <div class="archive-header">
            <span class="archive-classification">【机密·共鸣者档案】</span>
            <h2>共鸣状况检测报告</h2>
          </div>
          <span class="flame-icon"></span>
        </div>

        <div class="resonance-content">
          <!-- 报告编号 -->
          <div class="report-meta">
            <span class="report-id">报告编号：RS-2326-G</span>
            <span class="report-date">鉴定日期：瑝珑历·今州纪</span>
          </div>

          <!-- 报告主体（等宽字体） -->
          <div class="report-body">
            <h4>▍ 鉴定依据</h4>
            <p class="mono-text">【频谱检验报告RA2326-G】</p>

            <h4>▍ 共鸣频谱检验</h4>
            <p class="mono-text">
              无明确共鸣时间，主诉于幼年时期已出现操控火焰的能力，并多次出现控制不当的情况，成年后频率稳定性大幅提高。对象声痕位于胸部下方，共鸣后部分体征发生明显变化，主诉发梢与发尾褪色、局部出现翎羽状表征，左臂受共鸣能力影响出现朱红色灼痕并伴随火焰色光流。
            </p>
            <p class="mono-text">
              共鸣频谱检验图与火焰接近，测试有较强的谐振反应，无法完全确认共鸣源，暂未发现与该对象相似率超过60%的共鸣频谱检测图。其共鸣能力表征与瑝珑神话传说中的灵鸟类似，推测为对象幼年接触到相关内容影响所致。对象可将火焰凝聚为翎状武器，完全释放能力时催生温度极高、破坏范围极广的火焰，其表征为额前出现单眼雀翎状火焰，主诉称其为“心眼”。受试样本拉贝尔曲线呈稳定上升态，最终趋向稳定波动，检测结果判断为自然型共鸣者。
            </p>

            <h4>▍ 超频诊断报告</h4>
            <p class="mono-text">
              受试样本波形检测呈现椭圆形波动，时域表示稳定，未见异常波形。
            </p>

            <!-- 诊断标签组 -->
            <div class="diagnosis-badges">
              <span class="badge critical">超频临界值：高</span>
              <span class="badge stable">稳定性：高</span>
              <span class="badge safe">超频风险：无</span>
            </div>

            <p class="mono-text">
              存在过往超频史，最高等级：轻。超频时间主诉为童年期，曾长期处于高度危险的环境中，并多次因自卫或保护其他人员而进入爆发状态，致使对象受能力反噬影响，情况严重时危及生命。超频后遗症表征为左臂烧灼痕迹，频率波动随对象成长过程中对能力的控制力增强而趋于稳定，未有进一步扩散，但能力爆发造成的身体伤害不可逆，或导致对象寿命大幅缩短。
            </p>

            <!-- 医师建议 -->
            <div class="medical-note">
              <span class="note-icon">⚕️</span>
              <span class="note-text"
                >建议按时进行身体检查，暂无需心理辅导。</span
              >
            </div>
          </div>

          <!-- 档案页脚装订线 -->
          <div class="archive-footer">
            <span class="seal-stamp">今州共鸣研究所</span>
            <span class="seal-mark">【电子签章】</span>
          </div>
        </div>
      </section>

      <!-- 珍贵之物展示区（优化网格布局） -->
      <section class="treasures-grid">
        <div
          class="card treasure-card"
          v-for="(item, index) in treasureItems"
          :key="index"
          :style="{ '--card-index': index }"
        >
          <div class="treasure-img">
            <img
              :src="item.img"
              :alt="item.title"
              loading="lazy"
              decoding="async"
            />
            <div class="img-flame"></div>
          </div>
          <div class="treasure-info">
            <h3>{{ item.title }}</h3>
            <p v-for="(line, i) in item.description" :key="i">{{ line }}</p>
          </div>
        </div>
      </section>

      <!-- 新增：长离故事时间线板块（核心新增） -->
      <section
        id="timeline"
        class="card timeline-section"
        aria-labelledby="timeline-heading"
      >
        <div class="card-head">
          <h2 id="timeline-heading">长离纪事 · 时间线</h2>
          <span class="flame-icon"></span>
        </div>
        <div class="timeline-container">
          <div class="timeline-line"></div>
          <div class="timeline-items">
            <div
              v-for="(event, index) in timelineEvents"
              :key="index"
              class="timeline-item"
              :style="{ '--item-index': index }"
            >
              <div class="timeline-marker">
                <span class="marker-flame"></span>
              </div>
              <div class="timeline-content">
                <div class="timeline-age">{{ event.age }}</div>
                <h3 class="timeline-title">{{ event.title }}</h3>
                <p class="timeline-desc">{{ event.description }}</p>
              </div>
            </div>
          </div>
        </div>
      </section>

      <!-- 共鸣链详解（优化样式） -->
      <section id="resonance-chain" class="card resonance-chain">
        <div class="card-head">
          <h2>共鸣链详解</h2>
          <span class="flame-icon"></span>
        </div>
        <div class="chain-grid">
          <div
            v-for="(chain, index) in resonanceChains"
            :key="index"
            class="chain-item"
            :style="{ '--chain-index': index }"
          >
            <div class="chain-number">第 {{ index + 1 }} 链</div>
            <h3 class="chain-title">{{ chain.title }}</h3>
            <p class="chain-desc">{{ chain.desc }}</p>
          </div>
        </div>
      </section>

      <!-- 长离纪事故事合集（保留原有切换功能，优化样式） -->
      <section class="ll-stories" aria-label="长离的故事合集" ref="storyRef">
        <div class="card" role="region" aria-labelledby="stories-title">
          <header class="card-head">
            <h2 id="stories-title" class="card-title">长离纪事 · 故事集</h2>
            <span class="flame-icon"></span>
          </header>

          <main class="card-body">
            <transition name="fade-slide" mode="out-in">
              <article
                :key="items[current].id"
                :id="`panel-${items[current].id}`"
                class="story"
                role="tabpanel"
                :aria-labelledby="`tab-${items[current].id}`"
              >
                <h3 class="story-head">{{ items[current].title }}</h3>
                <div class="story-content">{{ items[current].content }}</div>
              </article>
            </transition>
          </main>

          <footer class="card-foot">
            <button class="nav-btn" @click="prev" aria-label="上一则故事">
              <span class="btn-flame"></span> ‹
            </button>
            <div class="pager">{{ current + 1 }} / {{ items.length }}</div>
            <button class="nav-btn" @click="next" aria-label="下一则故事">
              <span class="btn-flame"></span> ›
            </button>
          </footer>
        </div>
      </section>
    </main>
  </div>
</template>

<script setup lang="ts">
import { ref } from "vue";

// 核心机制列表
const mechanismItems = ref([
  "【离火】：血量上方上限4点的能量条。长离根据离火点数提供热熔伤害加成。攒满4点【离火】后，重击释放特殊重击【焚身以火】",
  "【离火】获取方式：第4段平A、E技能【心眼·劫】以及变奏入场能为长离提供12秒的【心眼】buff",
  "【心眼】：持续期间，长离使用平A派生【心眼·征】，或是跳跃派生【心眼·冲】，命中敌人后获得一点【离火】。长离释放共鸣解放【离火照丹心】后，会直接获得4点【离火】",
  "【输出流程】：E/变奏入场-【心眼·冲】-跳跃打断-AAAA-【心眼·冲】-A下落攻击-AA-【心眼·征】-E-【心眼·征】-Z【焚身以火】",
  "【小技巧】：长离的【心眼·冲】可以用跳跃打断，因此单人连招中使用跳跃缩减时间",
  "【技能缩写】：普攻 = A，重击 = Z，共鸣技能 = E，共鸣解放 = R，声骸 = Q",
]);

import item1 from "./img/item1.png";
import item2 from "./img/item2.png";
import item3 from "./img/item3.png";
import food from "./img/food.png";
// 珍贵之物数据（使用占位图片路径，请替换为实际图片）
const treasureItems = ref([
  {
    title: "炽翎辣肉",
    img: food,
    description: [
      "长离喜爱的菜肴，比寻常做法加了更多辣椒。赤红的色泽令辣度肉眼可见，灼辣与鲜香仿佛要化作实体从碗中振翅而出。",
      "在明庭时鲜少有人尝试，而在虹镇，这是家家户户都会的家常菜，温暖滋味。",
    ],
  },
  {
    title: "《乘霄乌鹭集》",
    img: item1,
    description: [
      "一本陈旧的棋谱，记载着一位奇人在乘霄山中留下的传世对局。",
      "如今所寻之人已在面前，未完的棋局也得以继续，只要对弈者仍在，这本棋谱便会继续更新下去。",
    ],
  },
  {
    title: "火浣布",
    img: item2,
    description: [
      "材质未知的纺织物，触感柔软，据说能够浴火不焚，经火焰涤荡后的布料反而会更加洁净纯白。",
      "继任大典后，今州的新任令尹将它作为礼物赠予了自己的恩师。",
    ],
  },
  {
    title: "心火羽",
    img: item3,
    description: [
      "长离以心火所化的羽毛，共有一对，如今其中一枚已赠予了今州的贵客，另一枚则一直被长离贴身收藏。",
      "很久以前，在小女孩喜欢的枕边故事中，浴火而生的灵鸟会将最为珍贵的尾翎赠予心中重要的存在。",
    ],
  },
]);

// 新增：长离故事时间线数据（根据搜索结果整理）[citation:4][citation:7]
const timelineEvents = ref([
  {
    age: "幼年时期",
    title: "村落毁灭 · 成为孤儿",
    description:
      "长离所生活的村子被残象潮毁灭，她是被村民藏匿在防火地下室中的唯一幸存者。",
  },
  {
    age: "童年",
    title: "拜师玄渺",
    description:
      "一场雨所带来的机遇与长离的救世之心使她与玄渺结识，并拜其为师，得师赐名“长离”。",
  },
  {
    age: "少年",
    title: "流浪学艺",
    description:
      "玄渺仙逝后，长离开始摸爬滚打，偷学百家之长，悟百家之道，被书斋老板满街追打是常事。",
  },
  {
    age: "青年",
    title: "担任长史",
    description:
      "成为前明庭中政省长史，在任期间主持建立了“悲田院”，收容灾难中失所的流民与孤儿。[citation:4]",
  },
  {
    age: "今州时期",
    title: "成为今汐的老师",
    description:
      "自请下今州代理事务，教导年幼的今汐，协助她接过对一个孩子来说过于沉重的权柄。",
  },
  {
    age: "如今",
    title: "担任令尹参事",
    description:
      "现今州令尹参事，善于洞察并利用人性引敌入局，将自己置身棋局，与时间博弈，也与毁灭博弈。",
  },
]);

// 共鸣链数据
const resonanceChains = ref([
  {
    title: "隐我所思",
    desc: "施放共鸣技能赫羽三相或重击焚身以火时，长离的抗打断能力提升，且造成的伤害提升10%。",
  },
  {
    title: "循我所望",
    desc: "获得【离火】时，长离的暴击提升25%，持续8秒。",
  },
  {
    title: "据我所闻",
    desc: "共鸣解放离火照丹心造成的伤害提升80%。",
  },
  {
    title: "饰我所言",
    desc: "施放变奏技能后，队伍中的角色攻击提升20%，持续30秒。",
  },
  {
    title: "舍我所得",
    desc: "重击焚身以火倍率提升50%，造成的伤害提升50%。",
  },
  {
    title: "成我所谋",
    desc: "共鸣技能赫羽三相、重击焚身以火和共鸣解放离火照丹心攻击造成伤害时额外忽视目标40%防御。",
  },
]);

// 故事合集数据（保留原有，稍作精简）
const items = ref([
  {
    id: "legend",
    title: "传言之外",
    content: `近两年来到边庭的人总说，很少见到那个传说中的参事长离。

虽说这位参事大人鲜少在公开场合露面，但作为令尹最信任的佐臣，她的名号在今州无人不知，而关于她的传闻，却千奇百怪，各有说法。

据说，长离大人在明庭时奏请修葺了各地的悲田院，令那些在悲鸣带来的灾难中失所的流民有了更好的归处；她还进谏中央庭减轻今州的商税，拨发粮款，助今州从弯刀之役中的阴影中恢复民生。

据说，长离大人刚到边庭时就不动声色地解决了两个盘踞在今州周边已久的流寇势力，仅是一场交谈，一个“显而易见的陷阱”便令两方互相猜忌缠斗，两败俱伤。

据说，长离大人并非岁主选定令尹之后才任命参事，而是一早便自请下今州代理事务，教导尚年幼的今汐，协助她接过对一个孩子来说过于沉重的权柄。

“真的假的？我怎么听说，前面那些都是上一任长史的举措，她是因为得罪了大人物，才被贬谪到边关来？"

“听说她是玄渺真人的亲传弟子，又得上一任长史举荐，游说四州，无人不拜服，怎么可能是个佞臣呢。”

“什么呀，我听说她是某个隐士大家的后裔，这次来今州，是因为肩负着家族传承的重任……”

诸如此类讨论甚嚣尘上，长离却似无所觉，偶尔撞见有人议论，她也只是毫不避讳地望过去，目光温和，却仿佛能看穿所有人的想法，反倒让那些人噤了声。

今州令尹由角选定，竟是个看上去尚未及笄的小姑娘，于是流言便更加猖獗，有人认为她手眼通天，盯上了令尹的地位，来此的目的就是为了控制那个年幼的孩子，取而代之。

而继任大典之后，长离便将理政之权完全交由今汐，被今汐任命为令尹参事。三年时间，今州政通人和，民众无不感念令尹勤政为民，这之中有多少是参事从旁裨辅，也从未有人知晓。

见到过长离的人都说，她总是带着温和的微笑，待人接物上至将领下至走卒，态度皆一视同仁，但独处时总是有种不怒自威的气势，令人有些不敢靠近。

“上次在巡宁所遇见长离大人，发现她一直饶有兴趣地盯着院中的树看，我原本以为她有什么要事在身，却没想到她只是在看蚂蚁搬家……看了整整一个下午……”

“然后呢？”

“后来有个被通缉很久的嫌犯头目突然过来自首，吓了我们一跳，起初以为又是什么陷阱，结果那人看到长离大人就像是受到了什么巨大的惊吓，一下子把什么都招了，而且还牵连出许多陈年疑案，这是我入行这么久，接过的最大的一个案子……”

“那长离大人呢？”

“不知什么时候离开了，唔，真是一位让人捉摸不透的大人啊……”`,
  },
  {
    id: "teacher",
    title: "师与臣",
    content: `在长离成为今汐的老师之前，曾与年幼的汐有过一面之缘。

那时长离前往今州，在边庭中行走时偶然听到侧殿中传来隐约的诵读声，有一个年幼的女孩正坐在比她还高的书堆里，努力地记忆那些对她的年纪来说有些艰深的理论。

“心不知治乱之源者，不可令制法……但要怎样才能做到有独闻之耳，独见之明呢。”

女孩懵懵懂懂地念出书中的内容，却有些不得其意。

那个小小的背影令长离想起了曾经的自己，也是这样努力地汲取着尽可能多的知识，着急地想要长大。

她不由得俯下身，为女孩解答她的疑问。

“汐受益匪浅。”女孩用稚嫩的嗓音一字一句地道谢，犹豫半响，又抬起头问她，“……那么，要怎样才能成为独当一面的令尹？”

“为何这样问？”

“今州人民苦于战事……我想看到，清平盛世。”

“……这并非易事。”

“我明白，但我想尽己所能。”

——我想看到，盛世太平。

长离看着女孩澄澈的双眼，似乎听到了星火燃烧的声响，微小却坚定。

“那么，我会助你达成所愿。”

曾经的小女孩被今州的岁主选定为新的令尹，边庭彻夜亮灯，长离始终陪在今汐身边，作为老师解答疑问，作为参事进言献策。

对于普通人来说，英雄的故事宏大而遥远，餐桌上的热饭热菜与能够安眠的夜晚才是日子变好的象征。

“汐，为政不可只高居庙堂，囿于案牍之间，要走到民众中去，聆听他们的心声。”

“汐明白。”

于是一件件柴米油盐中的愿望，寄托于今州许愿的习俗，汇聚为一个州城向好转变的开始。

长离看着今汐被热情的民众簇拥，萧索之态从他们的脸上尽数褪去，取而代之的是真心实意的幸福与感激。

今州已得明主，不论是民众还是中央庭，想必都不会再质疑今汐的能力。

“想到什么，便去做吧。”

“还有我在。”

老师，是播种之人，燃烛引路，令星火之光照亮天际。

她知道，这个眉宇间仍显稚嫩的女孩，会成长为足以庇护今州全境的，独当一面的令尹。

而她，会接过传承的星火，为自己的学生铺平前行的道路。`,
  },
  {
    id: "alley_full",
    title: "巷中孤羽",
    content: `延庆斋的常庆老板最近遇到了两件奇事。

一是书店的生意。

瑝珑地大物博，城中的时兴之物总在变化，近些年来人们不再推崇纸质书籍，书店的生意每况愈下。而这几日生意居然奇迹般地回暖，前来店中购书的人也比往常多上许多。

看装束，顾客多为中央庭官员——常庆心中疑惑，便去问店中顾客，得到的答复大多是：“这是长史大人推荐的地方”。

长史？他隐约听说最近中央庭新上任了长史，可他一个小小的书店老板，如何与这高居庙堂的大人有什么联系？

二是书店的门面。

生意萧条下，延庆斋的门面也许久未曾修葺，就连隔壁杂货铺的老板看了都直摇头。

“老常啊，你这门面确实该修整修整了，再破下去，别人真要把你这当烂仓库了！”

“你才烂仓库，虽说规模比不上明庭的书店，但论珍善孤本，我这延庆斋可是最全的！酒香不怕巷子深懂不懂。”

话虽这么说，但常庆心里也犯起了嘀咕，可他一把老骨头干不了重活，干脆趁最近生意回暖，雇个人修理也罢。

可第二天到店时他却惊得愣在了原地，原本陈旧落灰的门面一夜之间不复存在，牌匾与正门焕然一新，描金的“延庆斋”三字在阳光下闪闪发亮。

柜台前放着一个信笺，上面写着感念老板恩情，擅自修缮，聊表谢意。

信笺没有署名，但纸页上朱红色的火羽纹饰却有些眼熟，常庆攥着信笺缓缓坐了下来，陷入了回忆。

那还是在十几年前书店生意兴隆的时候，来来往往的顾客中总有个八九岁的小女娃悄悄溜进店里，捧着书一看就是一下午。

常庆起初不甚在意，开书店嘛，遇上只看不买的客人也是常事，只要不毁书不影响生意就成。

然而连着几日，销量断崖下跌，连进店的客人都少了许多。

“太热了太热了，待不下去，走走走，去别的店看看。”

时值早秋，屋内却热得异常，常庆擦擦额间的汗，仔细找了一圈，才发现书架的阴影中蜷缩着一个瘦小的影子，正捧着书看得聚精会神。

她的面前竟漂浮着一团火，房间里的温度就随着她看书入迷时的神情变换，逐渐走高——常庆看到眼前一幕，顿觉急火攻心。

“你是哪来的熊孩子，想烧我的店？！”

火焰倏然熄灭，女孩被吼声吓得一个激灵，常庆眼睁睁地瞅着她紧攥的那片书页逐渐焦黄，像是要烧起来了——

“你你你……！”

女孩慌忙丢下手中的书，看着纸页上的焦痕，她幼小的脸庞上流露出慌乱而愧疚的神色。

“对，对不起……这里太暗了，我只是想照明……”

眼见自己珍藏的孤本被糟蹋，脑中又划过这几日持续走低的账目，常庆黑着脸抄起书架边的扫帚，作势要打。

想不到这娃娃看着瘦小却窜得比猴还快，常庆追出门去却只看到那火焰色的发尾在空中划过，小小的身影瞬间就挤进了人头攒动的集市中，留下他站在原地气得吹胡子瞪眼睛。

“别让我再看见你！”

“怎么了，火气这么大？”旁边杂货铺的老板笑呵呵地走出来，顺着他的目光看了过去，“哟，那不是那个会控火的小娃娃吗。”

“你认识？她刚刚差点把我店给烧了！”

“哪有那么夸张，那是个好孩子啊，前几天还帮我干了不少活，只为了要些馒头……唉，也是可怜，小小年纪孤身一人，单靠一双腿走到这里，听说她来的地方被残象潮袭击，一个人都没活下来啊，也不知道她是怎么撑下来的……”

“……你咋不早说？”

“你也没问啊！不过看这样子……她应该是不敢再回来了吧。”

小女孩的瘦弱的身影已经消失在了巷尾，坊间传来邻家孩童嬉戏笑闹的声音，常庆看着落在地上的书，心中有些不是滋味。`,
  },
  {
    id: "heyi_full",
    title: "赫奕归林",
    content: `延庆斋的生意又回到了往常的样子，书客依旧络绎不绝，再也没见过那个缩在书架间默默读书的小女孩。

但让常庆没有想到的是，一周之后，那个孩子居然又回来了。

“这是我所有的钱……不是偷的！是我赚的，不知道够不够赔……抱歉，弄坏了您的书……我想看的书，只有先生这里有，所以才……”

女孩咬着嘴唇，怯生生地将手中零零碎碎的贝币放在了柜台上。那瘦小的身板还没有柜台高，只能露出半个脑袋，女孩踮着脚的身体微微发抖，而一双明亮得有些倔强的眼睛仍然毫不逃避地注视着他。

“……罢了。”

已经过去这么些天，气也早都消了，常庆摆了摆手，“走吧走吧，钱也拿走，别影响我做生意。”

女孩没有再说话，鞠了一躬后一步三回头地离开，看着女孩瘦弱的背影和破旧的衣衫，常庆沉默片刻，又重重叹了口气。

他起身拎着一盏灯走了过去，和店铺的备用钥匙一起塞到了女孩的手里。

“拿好，也别再用你那火照明了，晚上没客人的时候光明正大地来看。”

“……”

”没让你白看，早上顺便帮我把店给开了。”

“谢，谢谢您……”

那是他第一次见到那个女孩绽放出纯粹的，属于一个孩子的笑容。

那之后，女孩便时常来到延庆斋，她依照约定只在傍晚出现，一来便默默缩在角落读书，到清晨常庆来进店时，总是能看到店中被打扫的一尘不染，井井有条。

到了冬日，女孩便也能在白天来读书了。窗外寒风萧瑟，室内却温暖如春，常庆偶尔会撇一眼女孩在看什么，发现尽是一些说文识字、图解史记之类的内容。

那不甚符合年龄的沉静面庞，也只有在沉浸于书中时才有神采飞扬的时刻。

——后来的某天，女孩不知去向，有人说她遭到了残象袭击，有人说她伤重不治，死在了前往明庭的路上。

信笺上的火羽纹样在常庆眼中，愈发像是女孩曾经点起的那团火焰。

如果她还活着……

“老先生？”

常庆从沉思中回神，便看到柜台前站着一个女子，她身长玉立，眸光温和，火焰色的发尾如同传说中灵鸟的尾羽，正随晨风悠悠飘扬。

她将手中的贝币放在柜台，含笑开口。

“我想找的书，只有先生这里有，不知可否借读一日？”

——她不再像那个瘦弱的小女孩了，身姿气度倒是与街头巷尾议论纷纷的，那位官拜长史的年轻人如出一辙。

日头升上屋檐，带来一室晨光，年迈的书店老板笑了起来，摆了摆手，又靠回躺椅中。

“看吧，看吧……就算不给钱，我也赶不动你咯。”`,
  },
  {
    id: "tome_full",
    title: "典籍中的背影",
    content: `师父似乎一直在追随一个人。

年幼的长离跟着年迈的师父在瑝珑中游历，德高望重的老者本已不打算再授徒讲学，但既已收下了这小徒弟，便要尽心在最后的时间里将全部学识倾囊相授。

老人喜爱弈棋，在赶路疲乏的时候，师徒便席地而坐，摆出棋盘对弈。师父大多时间与长离下指导棋，在棋中教导她如何筹策万类，度权量能，但偶尔也会与她复盘一局未完的棋局，每每进行到中盘，师父却不甚满意。

到这时，他总会提起那个与他对弈的旧友，长离明白那人便是师父一直在追随的人，师父收她为徒，也是为了寻找这个人。但若是闻及那人姓甚名谁、现在何处，师父却总是摇摇头。

“时机未到。”

在预感到自己大限将至的那一日，老人将随身携带的棋谱与手记交给了长离，也将遗志传给了自己的徒儿。

“一切因果，皆在棋中。”

于是徒弟接过了师父的遗志，继续前行。

“……斯天人降世，乾坤指掌，纳骸于身。”

“……见金瞳异人于云霞明灭之间。”

“……与角并肩而立，亲密无间。”

她顺着师父留下的手记，一点点向前追寻，发现许多地方都存在着那个人的影子，有时是说书人口中精彩纷呈的故事，有时是史书典籍中只言片语的记载——那个存在如同坚定的基石与灯塔，在数度风雨飘摇的文明中将人们引向更好的未来。

那个人在世界上留下的痕迹，如同一缕难以察觉却坚韧绵长的丝线，将比普通人的寿命漫长得多的岁月与历史一一串联。

“那个特殊之人的出现，势必伴随着足以令你注意到的征兆。那个人，是真正有能力改变这个世界的人。”

而若想达成盛世太平的夙愿，却不能仅靠遥远的幻影，她必须站在足够高足够远的地方，才能有足够强的力量亲手触碰她追寻的未来，完成一路走来所有人托付于她的祈盼与约定，于是她游学四国，雪案萤窗，即使路途遥远艰险，也未曾放弃分毫。

官拜长史之后，她终于获得权限，如愿踏入瑝览类书，在浩如烟海的记载中，她找到了最为关键的信息。

信息所指向的，正是瑝珑的边陲重地，今州。

一切都如同岁主预言那般，贵客造访今州，破局止战，护得今州安宁——曾经只在典籍与传说中出现的模糊背影，如今终于有了确切的形态。

正式相见的那天，她从明庭匆匆赶回今州，见那位贵客正站在渡口，似乎在雨中踌躇。

——只要那位贵客按照角所言前行，我们总会相见。

如今，时机已到。

于是她隐去眼中如潮的情绪，一步一步走向那个人身边。

绘着灵鸟纹样的纸伞轻斜，二人四目相对，天地间只余细雨敲击伞面的声响。

她露出微笑，轻声开口。

“——你是被雨困在了这里吗？”`,
  },
]);

const current = ref(0);

const storyRef = ref<HTMLElement | null>(null);

function prev() {
  current.value = (current.value - 1 + items.value.length) % items.value.length;

  storyRef.value?.scrollIntoView({ behavior: "smooth", block: "start" });
}

function next() {
  current.value = (current.value + 1) % items.value.length;

  storyRef.value?.scrollIntoView({ behavior: "smooth", block: "start" });
}

// 粒子样式生成
function particleStyle(index: number) {
  const left = Math.random() * 100;
  const width = 2 + Math.random() * 8;
  const height = 15 + Math.random() * 30;
  const duration = 5 + Math.random() * 15;
  const delay = Math.random() * 8;
  const color = Math.random() > 0.5 ? "255, 140, 60" : "255, 80, 30";
  return {
    left: left + "%",
    width: width + "px",
    height: height + "px",
    animationDuration: duration + "s",
    animationDelay: delay + "s",
    backgroundColor: `rgba(${color}, 0.25)`,
  };
}
</script>

<style scoped lang="scss">
.changli-page {
  /* 长离主题色 */
  --deep-bg: #0a0503;
  --accent: #ff4f1e;
  --accent-2: #ff974f;
  --accent-3: #ffcb9a;
  --text-light: #f7e4d4;
  --glass-bg: rgba(10, 5, 3, 0.7);
  --flame-glow: rgba(255, 80, 20, 0.3);

  position: relative;
  min-height: 100vh;
  background: linear-gradient(135deg, #1a0c06 0%, #0a0503 50%, #1f0e07 100%);
  font-family: "Noto Sans SC", system-ui, -apple-system, sans-serif;
  color: var(--text-light);
  padding: clamp(20px, 4vw, 60px) clamp(16px, 3vw, 40px);
  overflow-x: hidden;

  /* 背景粒子层 */
  .bg-particles {
    position: fixed;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    pointer-events: none;
    z-index: 0;
    .particle {
      position: absolute;
      bottom: -20px;
      border-radius: 50%;
      filter: blur(6px);
      opacity: 0.2;
      animation: float-up 10s infinite ease-in-out;
    }
  }

  /* 底部火焰光带 */
  .page-glow {
    position: fixed;
    bottom: 0;
    left: 0;
    width: 100%;
    height: 4px;
    background: linear-gradient(
      90deg,
      transparent,
      var(--accent),
      var(--accent-2),
      var(--accent),
      transparent
    );
    filter: blur(6px);
    opacity: 0.5;
    animation: glow-pulse 4s infinite alternate;
    z-index: 1;
    pointer-events: none;
  }

  /* 棋盘背景装饰 */
  .board-bg {
    position: fixed;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    background-image: linear-gradient(
        rgba(255, 80, 30, 0.03) 1px,
        transparent 1px
      ),
      linear-gradient(90deg, rgba(255, 80, 30, 0.03) 1px, transparent 1px);
    background-size: 40px 40px;
    pointer-events: none;
    z-index: 0;
  }

  /* 翎羽装饰 */
  .feather-decoration {
    position: fixed;
    width: 200px;
    height: 400px;
    background: radial-gradient(
      ellipse at 50% 50%,
      rgba(255, 140, 60, 0.08),
      transparent 70%
    );
    filter: blur(40px);
    pointer-events: none;
    z-index: 0;
    animation: feather-float 12s infinite alternate;

    &.left {
      left: -100px;
      top: 20%;
      transform: rotate(15deg);
    }
    &.right {
      right: -100px;
      bottom: 20%;
      transform: rotate(-15deg);
    }
  }

  .container {
    position: relative;
    z-index: 10;
    max-width: 1200px;
    margin: 0 auto;
    padding-top: 60px;
  }

  /* ========== HERO 区域 ========== */
  .hero {
    position: relative;
    display: grid;
    grid-template-columns: 300px 1fr;
    gap: 32px;
    align-items: start;
    margin-bottom: 32px;
    z-index: 20;

    .left-col {
      display: flex;
      flex-direction: column;
      align-items: center;
    }

    .avatar-card {
      position: relative;
      width: 100%;
      background: rgba(20, 8, 4, 0.6);
      backdrop-filter: blur(10px);
      border-radius: 24px;
      padding: 16px;
      border: 1px solid rgba(255, 140, 60, 0.3);
      box-shadow: 0 20px 40px rgba(0, 0, 0, 0.5),
        0 0 30px rgba(255, 80, 20, 0.2);
      transition: transform 0.3s;

      &:hover {
        transform: translateY(-8px);
        box-shadow: 0 30px 60px rgba(0, 0, 0, 0.6),
          0 0 50px rgba(255, 100, 30, 0.3);
      }

      .avatar-flame {
        position: absolute;
        top: -10px;
        left: 50%;
        transform: translateX(-50%);
        width: 60px;
        height: 60px;
        background: radial-gradient(
          circle,
          rgba(255, 140, 60, 0.8),
          transparent 70%
        );
        filter: blur(15px);
        animation: flame-rotate 4s infinite alternate;
      }

      .avatar-wrap {
        width: 100%;
        border-radius: 16px;
        overflow: hidden;
        mask-image: linear-gradient(to bottom, black 80%, transparent);
      }

      .avatar {
        width: 100%;
        height: auto;
        display: block;
        transition: transform 0.5s;
      }

      &:hover .avatar {
        transform: scale(1.02);
      }

      .avatar-meta {
        margin-top: 12px;
        display: flex;
        flex-wrap: wrap;
        gap: 8px;
        justify-content: center;
      }

      .avatar-badge {
        background: rgba(255, 100, 30, 0.15);
        border: 1px solid rgba(255, 140, 60, 0.3);
        padding: 6px 12px;
        border-radius: 30px;
        font-size: 0.85rem;
        color: var(--accent-2);
        backdrop-filter: blur(5px);
      }
    }

    .right-col {
      display: flex;
      flex-direction: column;
    }

    .hero-panel {
      position: relative;
      background: rgba(10, 5, 3, 0.6);
      backdrop-filter: blur(12px);
      border-radius: 24px;
      padding: 32px;
      border: 1px solid rgba(255, 140, 60, 0.3);
      box-shadow: 0 20px 40px rgba(0, 0, 0, 0.5);
      overflow: hidden;

      &::before {
        content: "";
        position: absolute;
        top: 0;
        left: 0;
        width: 100%;
        height: 2px;
        background: linear-gradient(
          90deg,
          transparent,
          var(--accent),
          var(--accent-2),
          var(--accent),
          transparent
        );
        filter: blur(2px);
        animation: flow 6s linear infinite;
      }
    }

    .name {
      font-size: clamp(2rem, 4vw, 3.2rem);
      margin: 0;
      background: linear-gradient(135deg, #ffb47b, #ff6324, #ffb47b);
      -webkit-background-clip: text;
      background-clip: text;
      color: transparent;
      -webkit-text-fill-color: transparent;
      line-height: 1.2;
      font-weight: 800;
      letter-spacing: 1px;

      .jp {
        font-size: 1rem;
        color: var(--accent-2);
        margin-left: 12px;
        font-weight: 400;
        opacity: 0.9;
      }
    }

    .subtitle {
      font-size: 1.2rem;
      color: var(--accent-2);
      margin: 16px 0 8px;
      font-style: italic;
    }

    .lead {
      line-height: 1.7;
      color: #f0dac8;
    }

    .hero-flame-line {
      margin-top: 24px;
      height: 2px;
      width: 100px;
      background: linear-gradient(90deg, var(--accent), transparent);
      animation: width-pulse 3s infinite alternate;
    }
  }

  /* ========== 通用卡片 ========== */
  .card {
    background: rgba(10, 5, 3, 0.6);
    backdrop-filter: blur(12px);
    border-radius: 24px;
    padding: 28px;
    margin-bottom: 28px;
    border: 1px solid rgba(255, 140, 60, 0.2);
    box-shadow: 0 15px 35px rgba(0, 0, 0, 0.5);
    transition: transform 0.3s, box-shadow 0.3s;

    &:hover {
      transform: translateY(-5px);
      box-shadow: 0 25px 50px rgba(0, 0, 0, 0.6),
        0 0 30px rgba(255, 100, 30, 0.2);
      border-color: rgba(255, 140, 60, 0.4);
    }

    .card-head {
      display: flex;
      justify-content: space-between;
      align-items: center;
      margin-bottom: 20px;
      border-bottom: 1px solid rgba(255, 140, 60, 0.2);
      padding-bottom: 12px;

      h2 {
        margin: 0;
        font-size: 1.5rem;
        color: var(--accent-2);
        font-weight: 600;
        letter-spacing: 1px;
      }

      .flame-icon {
        width: 24px;
        height: 24px;
        background: radial-gradient(circle, var(--accent), transparent);
        border-radius: 50%;
        filter: blur(8px);
        animation: flame-flicker 2s infinite alternate;
      }
    }
  }

  /* 核心机制列表 */
  .personality .traits {
    list-style: none;
    padding: 0;
    margin: 0;

    li {
      display: flex;
      align-items: flex-start;
      gap: 12px;
      padding: 14px 0;
      border-bottom: 1px dashed rgba(255, 140, 60, 0.2);
      font-size: 1rem;
      line-height: 1.6;
      animation: slide-in 0.5s ease both;
      animation-delay: calc(0.1s * var(--item-index));

      .flame-bullet {
        flex-shrink: 0;
        width: 8px;
        height: 16px;
        background: linear-gradient(180deg, var(--accent), var(--accent-2));
        border-radius: 4px;
        transform: skewX(-10deg);
        filter: blur(1px);
        box-shadow: 0 0 10px var(--accent);
      }
    }
  }

  .overview {
    background: linear-gradient(
      145deg,
      rgba(18, 10, 5, 0.85),
      rgba(10, 5, 2, 0.9)
    );
    border: 1px solid rgba(200, 120, 60, 0.25);
    position: relative;
    overflow: hidden;

    /* 背景宣纸纹理 */
    &::before {
      content: "";
      position: absolute;
      top: 0;
      left: 0;
      width: 100%;
      height: 100%;
      background-image: repeating-linear-gradient(
        45deg,
        rgba(255, 140, 60, 0.02) 0px,
        rgba(255, 140, 60, 0.02) 2px,
        transparent 2px,
        transparent 6px
      );
      pointer-events: none;
    }

    .card-head {
      .title-group {
        display: flex;
        align-items: center;
        gap: 16px;

        .title-ornament {
          width: 40px;
          height: 2px;
          background: linear-gradient(
            90deg,
            transparent,
            #ff974f,
            #ff6324,
            #ff974f,
            transparent
          );
          filter: blur(1px);

          &.left {
            transform: skewX(-15deg);
          }
          &.right {
            transform: skewX(15deg);
          }
        }

        h2 {
          font-family: "Noto Serif SC", "Times New Roman", serif;
          font-size: 1.5rem;
          font-weight: 600;
          color: #ffd9b0;
          letter-spacing: 4px;
          text-shadow: 2px 2px 4px rgba(0, 0, 0, 0.3);
        }
      }
    }

    .two-col {
      display: grid;
      grid-template-columns: 1fr 1fr;
      gap: 32px;
      position: relative;
      z-index: 2;
    }

    /* 公共区块头部样式 */
    .block-header {
      display: flex;
      align-items: center;
      gap: 12px;
      margin-bottom: 20px;
      border-bottom: 1px solid rgba(255, 140, 60, 0.2);
      padding-bottom: 8px;

      .block-number {
        font-family: "Noto Serif SC", serif;
        font-size: 1.2rem;
        font-weight: 700;
        color: #ff974f;
        opacity: 0.8;
        transform: rotate(-5deg);
      }

      h3 {
        margin: 0;
        font-family: "Noto Serif SC", serif;
        font-size: 1.2rem;
        font-weight: 500;
        color: #ffd9b0;
        letter-spacing: 2px;
      }

      .header-line {
        flex: 1;
        height: 1px;
        background: linear-gradient(90deg, #ff974f, transparent);
      }
    }

    /* 左侧：背景区块（线装书） */
    .background-block {
      position: relative;
      padding: 8px 20px 20px 16px;
      background: rgba(0, 0, 0, 0.15);
      border-radius: 8px 20px 20px 8px;
      border-left: 4px solid #ff974f;

      .scroll-container {
        max-height: 320px;
        overflow-y: auto;
        padding-right: 8px;

        &::-webkit-scrollbar {
          width: 4px;
        }
        &::-webkit-scrollbar-track {
          background: rgba(255, 140, 60, 0.1);
          border-radius: 4px;
        }
        &::-webkit-scrollbar-thumb {
          background: #ff974f;
          border-radius: 4px;
        }
      }

      .classical-text {
        font-family: "Noto Serif SC", "楷体", serif;
        font-size: 0.95rem;
        line-height: 1.9;
        color: #ecdacb;
        text-indent: 2em;
        margin: 0;

        .dropcap {
          float: left;
          font-size: 3.2rem;
          line-height: 1;
          margin-right: 8px;
          color: #ff974f;
          font-weight: 700;
          text-shadow: 2px 2px 4px rgba(255, 100, 30, 0.3);
        }
      }

      /* 装订线 */
      .binding-line {
        position: absolute;
        left: -2px;
        top: 10%;
        width: 2px;
        height: 80%;
        background: repeating-linear-gradient(
          180deg,
          #ff974f,
          #ff974f 4px,
          transparent 4px,
          transparent 8px
        );
        opacity: 0.4;
      }

      /* 古籍印章 */
      .seal-mark {
        position: absolute;
        bottom: 16px;
        right: 20px;
        font-family: "Noto Serif SC", serif;
        font-size: 0.8rem;
        color: #ff6324;
        border: 1px solid #ff6324;
        padding: 2px 8px;
        border-radius: 20px;
        transform: rotate(-5deg);
        opacity: 0.7;
        background: rgba(0, 0, 0, 0.2);
        backdrop-filter: blur(2px);
      }
    }

    /* 右侧：外貌区块（绢本质感） */
    .appearance-block {
      position: relative;
      padding: 8px 16px 20px 20px;

      .portrait-frame {
        position: relative;
        padding: 16px;
        border: 1px solid rgba(255, 140, 60, 0.2);
        background: rgba(20, 10, 5, 0.3);
        border-radius: 24px 8px 24px 8px;

        .frame-corner {
          position: absolute;
          width: 20px;
          height: 20px;
          border-style: solid;
          border-color: #ff974f;
          opacity: 0.4;

          &.top-left {
            top: -2px;
            left: -2px;
            border-width: 2px 0 0 2px;
          }
          &.top-right {
            top: -2px;
            right: -2px;
            border-width: 2px 2px 0 0;
          }
          &.bottom-left {
            bottom: -2px;
            left: -2px;
            border-width: 0 0 2px 2px;
          }
          &.bottom-right {
            bottom: -2px;
            right: -2px;
            border-width: 0 2px 2px 0;
          }
        }
      }

      .elegant-text {
        font-family: "Noto Serif SC", "宋体", serif;
        font-size: 0.95rem;
        line-height: 1.8;
        color: #e0cfc0;
        margin: 0;
        text-align: justify;
        font-weight: 300;
        font-style: italic;
        letter-spacing: 0.3px;
      }

      /* 羽毛笔装饰 */
      .quill-decoration {
        position: absolute;
        bottom: -10px;
        right: -10px;
        width: 60px;
        height: 60px;
        background: radial-gradient(
          circle,
          rgba(255, 140, 60, 0.15),
          transparent
        );
        filter: blur(15px);
        transform: rotate(15deg);
      }
    }

    /* 底部卷轴装饰 */
    .scroll-decoration {
      display: flex;
      align-items: center;
      justify-content: center;
      gap: 16px;
      margin-top: 24px;
      padding-top: 16px;
      border-top: 1px solid rgba(255, 140, 60, 0.15);

      .scroll-left,
      .scroll-right {
        width: 60px;
        height: 2px;
        background: linear-gradient(90deg, transparent, #ff974f, transparent);
      }

      .scroll-text {
        font-family: "Noto Serif SC", serif;
        font-size: 0.8rem;
        color: #ffb07c;
        letter-spacing: 4px;
      }
    }
  }

  /* ========== 共鸣状况 · 档案鉴定风格（精简版） ========== */
  .resonance {
    background: linear-gradient(
      165deg,
      rgba(8, 4, 2, 0.9),
      rgba(15, 8, 4, 0.95)
    );
    border: 1px solid rgba(200, 120, 60, 0.3);
    font-family: "Inter", "Noto Sans SC", system-ui, sans-serif;

    .card-head {
      .archive-header {
        .archive-classification {
          display: block;
          font-size: 0.7rem;
          letter-spacing: 2px;
          color: #ff974f;
          margin-bottom: 2px;
          font-weight: 400;
        }

        h2 {
          margin: 0;
          font-size: 1.4rem;
          font-weight: 600;
          color: #ffd9b0;
          letter-spacing: 1px;
        }
      }
    }

    .resonance-content {
      padding-right: 16px;

      &::-webkit-scrollbar {
        width: 6px;
      }
      &::-webkit-scrollbar-track {
        background: rgba(255, 140, 60, 0.1);
        border-radius: 10px;
      }
      &::-webkit-scrollbar-thumb {
        background: linear-gradient(180deg, #ff974f, #ff4f1e);
        border-radius: 10px;
      }
    }

    /* 报告元信息 */
    .report-meta {
      display: flex;
      justify-content: space-between;
      font-size: 0.8rem;
      color: #c0a28a;
      border-bottom: 1px dashed rgba(255, 140, 60, 0.3);
      padding-bottom: 12px;
      margin-bottom: 20px;
      font-family: "JetBrains Mono", monospace;
    }

    /* 报告主体 */
    .report-body {
      h4 {
        color: #ffb47b;
        font-size: 0.95rem;
        margin: 20px 0 8px;
        border-left: 3px solid #ff6324;
        padding-left: 10px;
      }

      .mono-text {
        font-family: "JetBrains Mono", "Fira Code", monospace;
        font-size: 0.8rem;
        line-height: 1.7;
        color: #e0cfc0;
        margin-bottom: 16px;
      }
    }

    /* 诊断标签组 */
    .diagnosis-badges {
      display: flex;
      flex-wrap: wrap;
      gap: 8px;
      margin: 20px 0;

      .badge {
        padding: 4px 10px;
        border-radius: 20px;
        font-size: 0.7rem;
        font-weight: 500;
        background: rgba(0, 0, 0, 0.4);
        border: 1px solid;

        &.critical {
          border-color: #ff974f;
          color: #ff974f;
        }
        &.stable {
          border-color: #4aff9e;
          color: #a0ffc0;
        }
        &.safe {
          border-color: #4a9eff;
          color: #a0c8ff;
        }
      }
    }

    /* 医师建议 */
    .medical-note {
      display: flex;
      align-items: center;
      gap: 8px;
      background: rgba(255, 100, 30, 0.1);
      border-radius: 30px;
      padding: 8px 16px;
      margin: 20px 0;
      border: 1px solid rgba(255, 140, 60, 0.2);

      .note-icon {
        font-size: 1.2rem;
      }

      .note-text {
        font-size: 0.85rem;
        color: #ffd9b0;
        font-style: italic;
      }
    }

    /* 档案页脚 */
    .archive-footer {
      display: flex;
      justify-content: space-between;
      margin-top: 24px;
      padding-top: 16px;
      border-top: 1px solid rgba(255, 140, 60, 0.2);
      font-size: 0.7rem;
      color: #a08570;

      .seal-stamp {
        font-family: "Noto Serif SC", serif;
      }

      .seal-mark {
        color: #ff6324;
        opacity: 0.8;
      }
    }
  }

  /* 动画 */
  @keyframes wave-pulse {
    0% {
      opacity: 0.6;
      height: 20%;
    }
    100% {
      opacity: 1;
      height: var(--h);
    }
  }

  /* 响应式 */
  @media (max-width: 768px) {
    .overview .two-col {
      grid-template-columns: 1fr;
      gap: 24px;
    }

    .overview .card-head .title-group h2 {
      font-size: 1.2rem;
    }

    .resonance .card-head .archive-header h2 {
      font-size: 1.2rem;
    }
  }

  /* 珍贵之物网格 */
  .treasures-grid {
    display: grid;
    grid-template-columns: repeat(2, 1fr);
    gap: 24px;
    margin-bottom: 28px;

    .treasure-card {
      background: rgba(10, 5, 3, 0.6);
      backdrop-filter: blur(12px);
      border-radius: 20px;
      padding: 20px;
      border: 1px solid rgba(255, 140, 60, 0.2);
      transition: all 0.3s;
      animation: card-appear 0.6s ease both;
      animation-delay: calc(0.15s * var(--card-index));

      &:hover {
        transform: translateY(-8px);
        border-color: rgba(255, 140, 60, 0.5);
        box-shadow: 0 20px 40px rgba(0, 0, 0, 0.5),
          0 0 30px rgba(255, 100, 30, 0.3);
      }

      .treasure-img {
        position: relative;
        width: 100%;
        height: 160px;
        display: flex;
        align-items: center;
        justify-content: center;
        margin-bottom: 16px;

        img {
          max-width: 100%;
          max-height: 100%;
          object-fit: contain;
          filter: drop-shadow(0 10px 20px rgba(0, 0, 0, 0.5));
          transition: transform 0.3s;
        }

        .img-flame {
          position: absolute;
          width: 100%;
          height: 100%;
          background: radial-gradient(
            circle at 50% 50%,
            rgba(255, 140, 60, 0.3),
            transparent 70%
          );
          filter: blur(20px);
          z-index: -1;
        }
      }

      .treasure-info {
        h3 {
          color: var(--accent-2);
          margin: 0 0 12px;
          font-size: 1.2rem;
        }

        p {
          margin: 8px 0;
          font-size: 0.95rem;
          line-height: 1.6;
          color: #d9caba;
        }
      }
    }
  }

  /* ========== 新增：时间线板块 ========== */
  .timeline-section {
    position: relative;
    overflow: hidden;

    .timeline-container {
      position: relative;
      padding: 20px 0;
    }

    .timeline-line {
      position: absolute;
      left: 50%;
      transform: translateX(-50%);
      width: 4px;
      height: 100%;
      background: linear-gradient(
        180deg,
        transparent,
        var(--accent),
        var(--accent-2),
        var(--accent),
        transparent
      );
      filter: blur(2px);
      border-radius: 4px;
      animation: line-pulse 3s infinite alternate;
    }

    .timeline-items {
      position: relative;
      display: flex;
      flex-direction: column;
      gap: 32px;
    }

    .timeline-item {
      position: relative;
      display: flex;
      align-items: flex-start;
      gap: 32px;
      width: 100%;
      opacity: 0;
      animation: timeline-appear 0.8s ease forwards;
      animation-delay: calc(0.2s * var(--item-index));

      &:nth-child(odd) {
        flex-direction: row-reverse;

        .timeline-content {
          text-align: right;
        }
      }

      .timeline-marker {
        position: relative;
        flex-shrink: 0;
        width: 60px;
        display: flex;
        justify-content: center;

        .marker-flame {
          width: 24px;
          height: 24px;
          background: radial-gradient(circle, var(--accent), var(--accent-2));
          border-radius: 50%;
          filter: blur(8px);
          animation: marker-pulse 2s infinite alternate;
          box-shadow: 0 0 20px var(--accent);
        }
      }

      .timeline-content {
        flex: 1;
        max-width: 45%;
        background: rgba(20, 10, 5, 0.6);
        backdrop-filter: blur(8px);
        border-radius: 20px;
        padding: 20px;
        border: 1px solid rgba(255, 140, 60, 0.2);
        transition: transform 0.3s;

        &:hover {
          transform: translateY(-5px);
          border-color: rgba(255, 140, 60, 0.5);
        }

        .timeline-age {
          display: inline-block;
          background: rgba(255, 100, 30, 0.2);
          padding: 4px 12px;
          border-radius: 30px;
          font-size: 0.85rem;
          color: var(--accent-2);
          margin-bottom: 12px;
          border: 1px solid rgba(255, 140, 60, 0.3);
        }

        .timeline-title {
          color: var(--accent-2);
          margin: 0 0 12px;
          font-size: 1.2rem;
        }

        .timeline-desc {
          margin: 0;
          line-height: 1.6;
          color: #d9caba;
        }
      }
    }
  }

  /* 共鸣链网格 */
  .resonance-chain {
    .chain-grid {
      display: grid;
      grid-template-columns: repeat(3, 1fr);
      gap: 20px;

      .chain-item {
        background: rgba(255, 255, 255, 0.02);
        border-radius: 16px;
        padding: 20px;
        border: 1px solid rgba(255, 140, 60, 0.1);
        transition: all 0.3s;
        animation: chain-appear 0.5s ease both;
        animation-delay: calc(0.1s * var(--chain-index));

        &:hover {
          background: rgba(255, 100, 30, 0.1);
          border-color: rgba(255, 140, 60, 0.4);
          transform: translateY(-4px);
        }

        .chain-number {
          font-size: 0.85rem;
          color: var(--accent);
          margin-bottom: 8px;
          opacity: 0.8;
        }

        .chain-title {
          color: var(--accent-2);
          margin: 0 0 12px;
          font-size: 1.1rem;
        }

        .chain-desc {
          margin: 0;
          font-size: 0.9rem;
          line-height: 1.6;
          color: #d4c4b5;
        }
      }
    }
  }

  /* 故事合集 */
  .ll-stories {
    .card-body {
      min-height: 300px;
    }

    .story {
      .story-head {
        color: var(--accent-2);
        font-size: 1.3rem;
        margin-bottom: 16px;
      }

      .story-content {
        line-height: 1.8;
        color: #ecdacb;
        white-space: pre-wrap;
      }
    }

    .card-foot {
      display: flex;
      justify-content: space-between;
      align-items: center;
      margin-top: 24px;
      padding-top: 16px;
      border-top: 1px solid rgba(255, 140, 60, 0.2);

      .nav-btn {
        position: relative;
        background: rgba(255, 100, 30, 0.15);
        border: 1px solid rgba(255, 140, 60, 0.3);
        color: var(--accent-2);
        width: 44px;
        height: 44px;
        border-radius: 50%;
        font-size: 1.5rem;

        display: flex;
        align-items: center;
        justify-content: center;
        transition: all 0.3s;
        overflow: hidden;

        .btn-flame {
          position: absolute;
          width: 100%;
          height: 100%;
          background: radial-gradient(
            circle,
            rgba(255, 140, 60, 0.4),
            transparent
          );
          filter: blur(10px);
          opacity: 0;
          transition: opacity 0.3s;
        }

        &:hover {
          background: rgba(255, 140, 60, 0.3);
          transform: scale(1.1);
          .btn-flame {
            opacity: 1;
          }
        }
      }

      .pager {
        color: var(--accent-2);
        font-size: 1rem;
      }
    }
  }

  /* 动画定义 */
  @keyframes float-up {
    0% {
      transform: translateY(0) scale(1);
      opacity: 0.2;
    }
    100% {
      transform: translateY(-200px) scale(2);
      opacity: 0;
    }
  }

  @keyframes glow-pulse {
    0% {
      opacity: 0.3;
      filter: blur(4px);
    }
    100% {
      opacity: 0.8;
      filter: blur(8px);
    }
  }

  @keyframes feather-float {
    0% {
      transform: rotate(10deg) scale(1);
      opacity: 0.3;
    }
    100% {
      transform: rotate(20deg) scale(1.2);
      opacity: 0.5;
    }
  }

  @keyframes flame-rotate {
    0% {
      transform: translateX(-50%) rotate(0deg) scale(1);
      opacity: 0.5;
    }
    100% {
      transform: translateX(-50%) rotate(20deg) scale(1.2);
      opacity: 0.8;
    }
  }

  @keyframes flow {
    0% {
      background-position: 0% 50%;
    }
    50% {
      background-position: 100% 50%;
    }
    100% {
      background-position: 0% 50%;
    }
  }

  @keyframes width-pulse {
    0% {
      width: 100px;
      opacity: 0.5;
    }
    100% {
      width: 150px;
      opacity: 1;
    }
  }

  @keyframes flame-flicker {
    0% {
      opacity: 0.5;
      filter: blur(5px);
    }
    100% {
      opacity: 1;
      filter: blur(10px);
    }
  }

  @keyframes slide-in {
    from {
      opacity: 0;
      transform: translateX(-20px);
    }
    to {
      opacity: 1;
      transform: translateX(0);
    }
  }

  @keyframes card-appear {
    from {
      opacity: 0;
      transform: translateY(30px);
    }
    to {
      opacity: 1;
      transform: translateY(0);
    }
  }

  @keyframes timeline-appear {
    from {
      opacity: 0;
      transform: translateY(30px);
    }
    to {
      opacity: 1;
      transform: translateY(0);
    }
  }

  @keyframes chain-appear {
    from {
      opacity: 0;
      transform: scale(0.95);
    }
    to {
      opacity: 1;
      transform: scale(1);
    }
  }

  @keyframes line-pulse {
    0% {
      opacity: 0.3;
      filter: blur(2px);
    }
    100% {
      opacity: 0.8;
      filter: blur(4px);
    }
  }

  @keyframes marker-pulse {
    0% {
      opacity: 0.6;
      transform: scale(1);
    }
    100% {
      opacity: 1;
      transform: scale(1.2);
    }
  }

  /* 响应式 */
  @media (max-width: 1024px) {
    .hero {
      grid-template-columns: 250px 1fr;
    }

    .resonance-chain .chain-grid {
      grid-template-columns: repeat(2, 1fr);
    }

    .timeline-section .timeline-item {
      &:nth-child(odd) .timeline-content,
      .timeline-content {
        max-width: 80%;
      }
    }
  }

  @media (max-width: 768px) {
    .hero {
      grid-template-columns: 1fr;
    }

    .two-col {
      grid-template-columns: 1fr;
    }

    .treasures-grid {
      grid-template-columns: 1fr;
    }

    .resonance-chain .chain-grid {
      grid-template-columns: 1fr;
    }

    .timeline-section {
      .timeline-line {
        left: 30px;
      }

      .timeline-item {
        flex-direction: row !important;
        gap: 16px;

        .timeline-marker {
          width: 40px;
        }

        .timeline-content {
          max-width: calc(100% - 70px) !important;
          text-align: left !important;
        }
      }
    }

    .feather-decoration {
      display: none;
    }
  }
}

.reduced-motion * {
  transition: none !important;
  animation: none !important;
}

/* 滚动条美化 */
::-webkit-scrollbar {
  width: 8px;
}
::-webkit-scrollbar-track {
  background: rgba(10, 5, 3, 0.5);
}
::-webkit-scrollbar-thumb {
  background: linear-gradient(180deg, var(--accent), var(--accent-2));
  border-radius: 10px;
}
</style>
