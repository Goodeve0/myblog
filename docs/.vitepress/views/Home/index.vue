<template>
  <div class="home flex h-screen w-screen items-center justify-center">
    <div
      class="-mt-10 sm:-mt-40 flex w-screen animate-scale-in-center flex-col px-4 sm:w-[626px]"
    >
      <img
        src="../img/avatar.jpg"
        class="block w-48 h-48 mx-auto rounded-full animate-bounce-border"
      />
      <div
        class="mt-6 flex w-full flex-col items-center rounded-lg bg-white/70 py-6 text-zinc-800 shadow shadow-black/40 backdrop-blur-sm relative"
        data-aos="fade-in"
        data-aos-duration="1000"
      >
        <div class="text-2xl font-bold sm:text-3xl">
          <span class="shake-hand inline-block">👋</span> Hi, I am
          <span
            class="bg-gradient-to-r from-pink-500 to-rose-500 bg-clip-text text-transparent"
            >eve</span
          >.
        </div>
        <p class="mt-2 sm:text-lg text-base">
          eve's Tech Blog / Front-End Developer
        </p>
        <p class="mt-1 text-black/50"></p>
        <div class="mt-4 flex gap-4">
          <div
            @click="router.go('./AboutMe')"
            class="button relative w-fit cursor-pointer rounded-full bg-white active:scale-95"
          >
            🎉 Welcome
          </div>
          <div
            @click="gotoGithub"
            class="button button-github relative inline-flex cursor-pointer rounded-full bg-white active:scale-95"
          >
            <RiGithubLine class="w-5" />
            Github
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { onMounted, ref, onBeforeUnmount } from "vue";
import { RiGithubLine } from "@remixicon/vue";
import { useRouter } from "vitepress";
import AOS from "aos";
import "aos/dist/aos.css";
const returnToTopRef = ref<HTMLElement | null>(null);

const router = useRouter();
const gotoGithub = () => {
  window.open("https://github.com/goodeve0");
};

onMounted(() => {
  returnToTopRef.value = document.querySelector(".VPLocalNav.empty.fixed");
  if (returnToTopRef.value) returnToTopRef.value.style.zIndex = "-1000";
  AOS.init();
});

onBeforeUnmount(() => {
  if (returnToTopRef.value) returnToTopRef.value.style.zIndex = "1000";
});
</script>

<style scoped>
.home {
  --primary-color: #ff6086;
  background-image: url("../img/background.jpg");
  background-size: cover;
  background-position: center;
  background-attachment: fixed;
  min-height: 100vh;
}

.animate-bounce-border {
  position: relative;
  animation: bounce 2s infinite, gradient-bounce 3s infinite;
  border: 2px solid transparent;
  border-radius: 50%;
  background-image: linear-gradient(45deg, #ff6086, #ff9f9f, #ff6086);
  background-size: 400% 400%;
  animation: bounce 2s infinite, gradient-bounce 3s ease infinite;
  box-shadow: 0 0 15px rgba(255, 255, 255, 0.6);
}

@keyframes gradient-bounce {
  0% {
    background-position: 100% 0;
  }
  50% {
    background-position: 0 100%;
  }
  100% {
    background-position: 100% 0;
  }
}

@keyframes bounce {
  0%,
  100% {
    transform: translateY(0);
  }
  50% {
    transform: translateY(-5px);
  }
}

@keyframes bounce {
  0%,
  100% {
    transform: translateY(0);
  }
  50% {
    transform: translateY(-5px);
  }
}

.button-github {
  --primary-color: #38bdf8;
}

.button {
  padding: 6px 12px;
  color: var(--primary-color);
  border: 2px solid var(--primary-color);
  transition: all 0.2s cubic-bezier(0, 0, 0.2, 1);
}

.button:hover {
  background-color: var(--primary-color);
  color: white;
}

.button::before,
.button::after {
  position: absolute;
  content: "";
  width: 150%;
  left: 50%;
  height: 100%;
  transform: translateX(-50%);
  z-index: -1000;
  background-repeat: no-repeat;
}

.button:hover::before {
  top: -70%;
  background-image: radial-gradient(
      circle,
      var(--primary-color) 20%,
      transparent 20%
    ),
    radial-gradient(
      circle,
      transparent 20%,
      var(--primary-color) 20%,
      transparent 30%
    ),
    radial-gradient(circle, var(--primary-color) 20%, transparent 20%),
    radial-gradient(circle, var(--primary-color) 20%, transparent 20%),
    radial-gradient(
      circle,
      transparent 10%,
      var(--primary-color) 15%,
      transparent 20%
    ),
    radial-gradient(circle, var(--primary-color) 20%, transparent 20%),
    radial-gradient(circle, var(--primary-color) 20%, transparent 20%),
    radial-gradient(circle, var(--primary-color) 20%, transparent 20%),
    radial-gradient(circle, var(--primary-color) 20%, transparent 20%);
  background-size: 10% 10%, 20% 20%, 15% 15%, 20% 20%, 18% 18%, 10% 10%, 15% 15%,
    10% 10%, 18% 18%;
  background-position: 50% 120%;
  animation: greentopBubbles 1s ease;
}

@keyframes greentopBubbles {
  0% {
    background-position: 5% 90%, 10% 90%, 10% 90%, 15% 90%, 25% 90%, 25% 90%,
      40% 90%, 55% 90%, 70% 90%;
  }
  50% {
    background-position: 0% 80%, 0% 20%, 10% 40%, 20% 0%, 30% 30%, 22% 50%,
      50% 50%, 65% 20%, 90% 30%;
  }
  100% {
    background-position: 0% 70%, 0% 10%, 10% 30%, 20% -10%, 30% 20%, 22% 40%,
      50% 40%, 65% 10%, 90% 20%;
    background-size: 0% 0%, 0% 0%, 0% 0%, 0% 0%, 0% 0%, 0% 0%;
  }
}

.button:hover::after {
  bottom: -70%;
  background-image: radial-gradient(
      circle,
      var(--primary-color) 20%,
      transparent 20%
    ),
    radial-gradient(circle, var(--primary-color) 20%, transparent 20%),
    radial-gradient(
      circle,
      transparent 10%,
      var(--primary-color) 15%,
      transparent 20%
    ),
    radial-gradient(circle, var(--primary-color) 20%, transparent 20%),
    radial-gradient(circle, var(--primary-color) 20%, transparent 20%),
    radial-gradient(circle, var(--primary-color) 20%, transparent 20%),
    radial-gradient(circle, var(--primary-color) 20%, transparent 20%);
  background-size: 15% 15%, 20% 20%, 18% 18%, 20% 20%, 15% 15%, 20% 20%, 18% 18%;
  background-position: 50% 0%;
  animation: greenbottomBubbles 1s ease;
}

@keyframes greenbottomBubbles {
  0% {
    background-position: 10% -10%, 30% 10%, 55% -10%, 70% -10%, 85% -10%,
      70% -10%, 70% 0%;
  }
  50% {
    background-position: 0% 80%, 20% 80%, 45% 60%, 60% 100%, 75% 70%, 95% 60%,
      105% 0%;
  }
  100% {
    background-position: 0% 90%, 20% 90%, 45% 70%, 60% 110%, 75% 80%, 95% 70%,
      110% 10%;
    background-size: 0% 0%, 0% 0%, 0% 0%, 0% 0%, 0% 0%, 0% 0%;
  }
}

.shake-hand {
  animation: shake 4s ease-in-out both infinite;
}

@keyframes shake {
  60% {
    transform: rotate(0deg);
  }
  70% {
    transform: rotate(-10deg);
  }
  80% {
    transform: rotate(0deg);
  }
  90% {
    transform: rotate(10deg);
  }
  100% {
    transform: rotate(0deg);
  }
}
</style>
