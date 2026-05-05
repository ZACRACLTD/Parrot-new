<script setup lang="ts">
import { onMounted, onUnmounted, ref } from "vue";
import gsap from "gsap";
import ScrollTrigger from "gsap/ScrollTrigger";

gsap.registerPlugin(ScrollTrigger);

const { hideNavbar, showNavbar } = useNavbar();

const tabContent: Record<
  string,
  { title: string; bullets: string[]; description: string; image: string }
> = {
  Verify: {
    title: "Verify if a Business is Legit Before Paying",
    description:
      "Share your honest experience. Your review posts directly to the business profile and stays there permanently.",
    bullets: [
      "Owners identity verification",
      "Bank account verification",
      "Social handles and address verification",
      "What customers love and complain about",
    ],
    image: "/images/Verify.png",
  },
  Reviews: {
    title: "Know What Past Customers Have Experienced",
    description:
      "Ask the community anything about a business, product, or service before you spend.",
    bullets: [
      "Get honest reviews in video, voice note, text.",
      "Message reviewers to ask more questions about their review",
      "Get detailed reviews from Pro Reviewers.",
    ],
    image: "/images/Reviews.png",
  },
  Share: {
    title: "Share your reviews openly and freely",
    description:
      "Search verified reviews from real customers you can actually trust.",
    bullets: [
      "Share honest reviews in videos, voice note, or text",
      "Report scams and bad customer experience.",
      "Help other customers avoid bad buys",
      " Help businesses know how to improve.",
    ],
    image: "/images/Share.png",
  },
  Ask: {
    title: "Ask and Get Honest Answers from the Community",
    description: "Verify any business before handing over your money.",
    bullets: [
      "Ask for product and business recommendations.",
      "Interest and location-based answers",
      "Real customers not influencers.",
    ],
    image: "/images/Ask.png",
  },
};

const tabs = Object.keys(tabContent);
const activeTab = ref(tabs[0]);
const cards = ref<HTMLElement[]>([]);
let tl: gsap.core.Timeline;

// How much each card peeks below the active one
const PEEK_Y = 6; // Reduced for mobile
const PEEK_SCALE = 0.02; // Reduced for mobile

onMounted(() => {
  cards.value = gsap.utils.toArray(".stack-card");
  const total = cards.value.length;

  // Set initial stacked state: card 0 on top, rest peeking below
  cards.value.forEach((card, index) => {
    gsap.set(card, {
      zIndex: total - index,
      yPercent: 0,
      y: index * PEEK_Y,
      scale: 1 - index * PEEK_SCALE,
      transformOrigin: "top center",
      opacity: index < 3 ? 1 : 0, // only show top 3 in stack
    });
  });

  tl = gsap.timeline({
    scrollTrigger: {
      trigger: ".stack-section",
      start: "top top",
      end: `+=${tabs.length * 800}`,
      scrub: 1.2,
      pin: ".stack-sticky",

      onUpdate: (self) => {
        const index = Math.min(
          total - 1,
          Math.round(self.progress * (total - 1)),
        );
        activeTab.value = tabs[index];
      },

      onEnter: hideNavbar,
      onEnterBack: hideNavbar,
      onLeave: showNavbar,
      onLeaveBack: showNavbar,
    },
  });

  cards.value.forEach((card, index) => {
    if (index === total - 1) return;

    const step = gsap.timeline();

    // 1. Top card exits: slides up and off screen
    step.to(
      card,
      {
        yPercent: -115,
        scale: 0.92,
        opacity: 0,
        duration: 1,
        ease: "power3.inOut",
      },
      0,
    );

    // 2. All remaining cards below animate forward into their new positions
    for (let j = index + 1; j < total; j++) {
      const newOffset = j - (index + 1); // how far below the new top card they'll be
      step.to(
        cards.value[j],
        {
          y: newOffset * PEEK_Y,
          scale: 1 - newOffset * PEEK_SCALE,
          opacity: newOffset < 3 ? 1 : 0,
          duration: 1,
          ease: "power3.inOut",
        },
        0, // all happen simultaneously
      );
    }

    tl.add(step, index);
  });
});

function goToTab(tab: string) {
  const index = tabs.indexOf(tab);
  activeTab.value = tab;

  const st = ScrollTrigger.getAll().find((t) =>
    t.trigger?.classList?.contains("stack-section"),
  );
  if (!st) return;

  const progress = index / (tabs.length - 1);
  gsap.to(window, {
    scrollTo: st.start + (st.end - st.start) * progress,
    duration: 0.7,
    ease: "power2.out",
  });
}

onUnmounted(() => {
  ScrollTrigger.getAll().forEach((t) => t.kill());
});
</script>

<template>
   <section class="stack-section relative md:h-[620vh] h-[500vh] text-black">
    <div
      class="stack-sticky h-screen flex flex-col gap-8 items-start justify-between overflow-hidden"
    >
      <!-- Tabs -->
      <div class="flex gap-2 flex-wrap mt-6">
        <button
          v-for="t in tabs"
          :key="t"
          :class="[
            'px-3 py-3 md:px-4 md:py-2 rounded-[8px] border-2 shadow-brutal-sm border-black transition-colors duration-300 text-sm md:text-base',
            activeTab === t ? 'bg-navy text-white' : 'bg-white text-black',
          ]"
          @click="goToTab(t)"
        >
          {{ t }}
        </button>
      </div>

      <!-- Stack container — extra bottom padding so peeking cards are visible -->
       <div
         class="relative w-full"
         style="height: calc(85vh + 36px); padding-bottom: 36px"
       >
        <div
          v-for="(tab, index) in tabs"
          :key="tab"
          class="stack-card absolute inset-x-0 md:top-0 top-10 md:h-[80vh] h-[65vh]"
          
        >
          <div
            class="h-full w-full rounded-3xl border-2 border-black bg-white p-8 md:p-16 shadow-[0_8px_40px_rgba(0,0,0,0.08)]"
          >
            <div class="flex flex-col md:flex-row gap-8 md:gap-16 h-full items-center">
              <div class="w-full md:w-4/12">
                <img
                  :src="tabContent[tab].image"
                  class="rounded-xl md:block hidden h-48 md:h-full w-full object-contain"
                />
              </div>
              <div class="w-full md:w-5/12 flex flex-col gap-4 md:gap-6">
                <h2 class="text-2xl md:text-4xl">
                  {{ tabContent[tab].title }}
                </h2>
                <p class="text-gray-600 text-sm md:text-base">
                  {{ tabContent[tab].description }}
                </p>
                <div class="flex flex-col gap-3 md:gap-4">
                  <div
                    v-for="bullet in tabContent[tab].bullets"
                    :key="bullet"
                    class="flex gap-3 items-center"
                  >
                    <div class="bg-white border p-2 shadow-brutal-sm rounded-[8px]  ">
                      <img src="/images/checkmark.svg" class="w-4 h-4 md:w-6 md:h-6" />
                    </div>

                    <span class="flex-1 text-sm md:text-base">{{ bullet }}</span>
                  </div>
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </section>
</template>
