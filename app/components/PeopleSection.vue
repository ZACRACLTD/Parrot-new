<script setup lang="ts">
import { onMounted, onUnmounted, ref, nextTick } from "vue";
import gsap from "gsap";
import ScrollTrigger from "gsap/ScrollTrigger";

gsap.registerPlugin(ScrollTrigger);

const { hideNavbar, showNavbar } = useNavbar();

const peopleContent: Record<
  string,
  {
    type: string;
    image: string;
    description: string;
    quote: string;
    cta: string;
  }
> = {
  "UGC Creators": {
    type: "UGC Creators",
    image: "/images/ugc-creator.png",
    description:
      "They post business and product discovery content in video, photo, and text — and they make it worth watching. Fun. Honest. Opinionated. Shareable.",
    quote:
      '"My video on the best Amala spot on the Island has been shared 800 times."',
    cta: "Become a UGC Creator",
  },

  "Everyday Customers": {
    type: "Everyday Customers",
    image: "/images/customers.png",
    description:
      "They bought something. They experienced something. They came here to tell the truth about it. They are the realest review you will ever read.",
    quote: '"Turns out, thirty people needed to hear what I had to say."',
    cta: "Join as a Customer",
  },

  "Pro Reviewers": {
    type: "Pro Reviewers",
    image: "/images/reviewers.png",
    description:
      "They go all the way in. They order the product. They visit the location. They test the service. They write the kind of review that leaves no questions unanswered.",
    quote:
      '"I spent two weeks with the DANG Hydra Glow Gel. You needed to know what I found."',
    cta: "Become a Pro Reviewer",
  },

  "Verified Businesses": {
    type: "Verified Businesses",
    image: "/images/business.png",
    description:
      "They post business and product discovery content in video, photo, and text — and they make it worth watching. Fun. Honest. Opinionated. Shareable.",
    quote:
      '"My video on the best Amala spot on the Island has been shared 800 times."',
    cta: "Claim Your Business",
  },
};

const tabs = Object.keys(peopleContent);
const activeTab = ref(tabs[0]);
const cards = ref<HTMLElement[]>([]);
let tl: gsap.core.Timeline;

const PEEK_Y = 12;
const PEEK_SCALE = 0.04;

onMounted(async () => {
  await nextTick();

  cards.value = gsap.utils.toArray(".people-card");
  const total = cards.value.length;

  // Initial stacked state: card 0 on top, rest peeking below
  cards.value.forEach((card, index) => {
    gsap.set(card, {
      zIndex: total - index,
      yPercent: 0,
      y: index * PEEK_Y,
      scale: 1 - index * PEEK_SCALE,
      transformOrigin: "top center",
      opacity: index < 3 ? 1 : 0,
    });
  });

  tl = gsap.timeline({
    scrollTrigger: {
      trigger: ".people-section",
      start: "top top",
      end: `+=${tabs.length * 1200}`,
      scrub: 1.2,
      pin: ".people-sticky",

      onUpdate: (self) => {
        const index = Math.min(
          total - 1,
          Math.round(self.progress * (total - 1))
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

    // Top card sweeps up and off
    step.to(
      card,
      {
        yPercent: -115,
        scale: 0.92,
        opacity: 0,
        duration: 1,
        ease: "power3.inOut",
      },
      0
    );

    // All cards below advance forward into new positions
    for (let j = index + 1; j < total; j++) {
      const newOffset = j - (index + 1);
      step.to(
        cards.value[j],
        {
          y: newOffset * PEEK_Y,
          scale: 1 - newOffset * PEEK_SCALE,
          opacity: newOffset < 3 ? 1 : 0,
          duration: 1,
          ease: "power3.inOut",
        },
        0
      );
    }

    tl.add(step, index);
  });
});

function goToTab(tab: string) {
  const index = tabs.indexOf(tab);
  activeTab.value = tab;

  const st = ScrollTrigger.getAll().find((t) =>
    t.trigger?.classList?.contains("people-section")
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
   <section class="people-section relative h-[880vh] text-black">
    <div
      class="people-sticky sticky top-0 h-screen flex items-center justify-center overflow-hidden"
    >
      <div
        class="relative  w-full max-w-7xl"
        style="height: calc(85vh + 36px); padding-bottom: 36px;"
      >
        <div
          v-for="(tab, index) in tabs"
          :key="tab"
          class="people-card absolute inset-x-0 top-0 rounded-3xl border-2 border-black bg-white shadow-brutal p-6 md:p-12"
          :style="{ height: '85vh' }"
        >
            <div class="flex flex-col md:flex-row gap-6 md:gap-12 h-full">
              <div class="w-full md:w-1/2">
                <img
                  :src="peopleContent[tab].image"
                  :alt="peopleContent[tab].type"
                  class="rounded-xl h-48 md:h-[400px] w-full object-cover shadow-brutal"
                />
              </div>

              <div
                class="w-full md:w-1/2 flex flex-col justify-center gap-4 md:gap-6"
              >
              <h3
                class="font-heading text-2xl md:text-3xl lg:text-5xl font-normal text-black"
              >
                {{ peopleContent[tab].type }}
              </h3>

              <div class="flex flex-col gap-4 md:gap-5">
                <p
                  class="font-body text-black text-sm md:text-lg leading-relaxed"
                >
                  {{ peopleContent[tab].description }}
                </p>

                <p
                  class="font-body font-bold text-black text-sm md:text-lg leading-relaxed"
                >
                  {{ peopleContent[tab].quote }}
                </p>
              </div>

              <a
                :href="
                  peopleContent[tab].cta === 'Claim Your Business'
                    ? 'https://parrot.id/claim-business'
                    : 'https://onelink.to/nwmnzy'
                "
                class="inline-flex items-center border-2 border-black bg-parrot-amber text-black font-body font-medium text-sm md:text-base px-5 md:px-6 py-2 md:py-2.5 rounded-lg hover:brightness-95 transition-all self-start"
              >
                {{ peopleContent[tab].cta }}
              </a>
            </div>
          </div>
        </div>
      </div>
    </div>
  </section>
</template>