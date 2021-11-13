<template>
  <div class="flex flex-col">
    <h1 class="text-4xl mb-16">
      Grading Helper <button @click="changeIcon()" v-text="icon"></button>
    </h1>

    <div class="mb-4 flex flex-col">
      <label
        for="total"
        title="The total number of points the student can earn on this assignment"
        >Points possible:</label
      >
      <div class="flex w-full">
        <input
          id="total"
          ref="totalRef"
          v-model.number="total"
          class="
            border
            rounded
            p-4
            flex-grow
            dark:text-gray-800 dark:bg-gray-200
            disabled:dark:bg-gray-400 disabled:cursor-not-allowed
            min-w-0
            max-w-full
          "
          type="text"
          inputmode="decimal"
          autocomplete="off"
          min="0"
          :class="{ invalid: errorMsg !== '' }"
          :disabled="locked"
          :readonly="locked"
          @blur="validateTotal()"
          @focus="errorMsg = ''"
        />

        <button
          class="p-2 rounded ml-2"
          :title="'Press to ' + (locked ? 'unlock' : 'lock')"
          @click="toggleLock()"
          v-text="!locked ? '🔒' : '✏️'"
        ></button>
      </div>
      <p class="text-sm text-red-600" v-text="errorMsg"></p>
    </div>

    <div class="mb-4 flex flex-col">
      <label
        for="points_missed"
        title="The number of points the student missed on the assignment."
        >Points missed:</label
      >
      <input
        id="points_missed"
        v-model.number="points_missed"
        class="
          border
          rounded
          p-4
          dark:text-gray-800 dark:bg-gray-200
          min-w-0
          max-w-full
        "
        min="0"
        type="text"
        inputmode="decimal"
        autocomplete="off"
        :max="total"
      />
    </div>

    <div class="mt-16">
      <div v-if="isNaN(score) || !isFinite(score)">
        <h2 class="font-bold">Enter values above to calculate</h2>
      </div>
      <div v-else-if="score < 0 || score > 100">
        <h2 class="font-bold text-red-800 text-sm">
          Score is not between zero and one hundred
        </h2>
      </div>
      <div v-else>
        <h2 class="font-bold">
          Final Score:
          <span
            class="pl-4 circled"
            :title="score"
            v-text="score.toFixed(0) + '%'"
          ></span>
        </h2>
      </div>
    </div>
  </div>
</template>

<script lang="ts">
import { Options, Vue } from "vue-class-component";

const ICONS = [
  "🍎",
  "📝",
  "🍏",
  "👨‍🎓",
  "🧑‍🎓",
  "👩‍🎓",
  "👩‍💻",
  "☕",
  "🎱",
  "🧘",
  "😤",
  "😭",
  "🤣",
  "👩‍🏫",
  "🎒",
  "📚",
  "💯",
];

@Options({
  watch: {
    locked(newLocked) {
      if (newLocked) {
        localStorage.total = this.total;
      } else {
        localStorage.removeItem("total");
      }

      localStorage.locked = newLocked ? 1 : 0;
    },
  },
})
export default class HelloWorld extends Vue {
  total = 0 as number | string;
  points_missed = 0 as number | string;
  locked = false;
  icon = "";
  errorMsg = "";

  $refs!: {
    totalRef: HTMLInputElement;
  };

  mounted(): void {
    if (localStorage.total) {
      this.total = localStorage.total;

      // Only lock if we have a total
      if ("locked" in localStorage) {
        this.locked = localStorage.locked && localStorage.locked !== "false";
      }
    }

    this.randomlyChangeIcon();
  }

  get score(): number {
    if (typeof this.total === "string") {
      return NaN;
    }

    if (typeof this.points_missed === "string") {
      return NaN;
    }

    return ((this.total - this.points_missed) / this.total) * 100;
  }

  changeIcon(): void {
    this.icon = ICONS[Math.floor(Math.random() * ICONS.length)];
  }

  randomlyChangeIcon(): void {
    this.changeIcon();

    setTimeout(this.randomlyChangeIcon, Math.random() * 30000);
  }

  toggleLock(): void {
    this.validateTotal();

    if (this.errorMsg === "" || this.locked) {
      this.locked = !this.locked;
    }
  }

  validateTotal(): void {
    let errorMsg = "";

    if (typeof this.total === "string") {
      errorMsg = "Please enter a number";
    } else if (isNaN(this.total) || !isFinite(this.total)) {
      errorMsg = "Please enter a valid number";
    } else if (this.total < 0) {
      errorMsg = "Please enter a number greater than zero";
    }

    this.$refs.totalRef.setCustomValidity(errorMsg);
    this.errorMsg = errorMsg;
  }
}
</script>

<style scoped>
#total.invalid {
  @apply bg-red-200;
}

.circled {
  background: url("../assets/circled.png") no-repeat center center;
  background-size: contain;
  padding: 35px 30px 25px;
}
</style>
