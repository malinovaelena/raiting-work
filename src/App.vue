<template>
  <div id="app">
    <div class="container d-flex col-12">
      <div class="task-condition text-left col-4">
        <div>объем оперативной памяти – 256 Мбайт,</div>
        <div>количество разделов до10,</div>
        <div>очередь задач – общая,</div>
        <div>размер задачи – случайный – от 30 до 100 Мбайт,</div>
        <div>количество задач в очереди -20.</div>
        <button class="button m-3" @click="run">запустить</button>
      </div>
      <div class="os-progress col">
        <div class="OS">
          <div
            v-for="(part, ind) in RAMSections"
            :key="ind * Math.random"
            class="os-part"
            :class="{ 'os-part--busy': !part.free }"
            :style="widthSize(part)"
          ></div>
        </div>
        <div class="messages w-100 h-100 text-left">
          <div
            v-for="(message, key) in consoleMessages"
            :key="key * Math.random"
            class="message w-100"
          >
            {{ message }}
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
const MESSAGE = {
  free: "Память полностью свободна",
  busy: "Недостаточно места",
};

export default {
  name: "App",

  data() {
    return {
      consoleMessages: [],
      tasksList: [],
      RAMSections: [],
      RamVolume: 256,
      maxAmountSections: 10,
      minTaskVolume: 30,
      maxTaskVolume: 100,
      message: MESSAGE,
      amountTask: 20,
    };
  },

  created() {
    this.initTasks();
    this.initSections();
  },

  mounted() {
   // this.main();
  },

  methods: {
    run() {
      this.consoleMessages = [];
      this.RAMSections = [];
      this.initSections();
      this.main();
    },
    widthSize(part) {
      return `width: ${part.size / (256 / 100)}%`;
    },
    initSections() {
      for (let i = 0; i < this.maxAmountSections; i++) {
        this.RAMSections.push({
          size: this.RamVolume / this.maxAmountSections,
          free: true,
          order: -1,
        });
      }
    },
    initTasks() {
      for (let j = 1; j < this.amountTask; j++) {
        let r = this.random(this.minTaskVolume, this.maxTaskVolume);
        this.tasksList.push({
          size: r,
          order: j,
        });
      }
    },
    main() {
      let allFree = 0;
      //if (!window.confirm("Начать выполнение программы ?")) return;
      this.tasksList.forEach((item, ind) => {
        const start = this.RAMSections.findIndex((r) => r.free);
        const index = this.findFree(this.RAMSections, 0, item.size);

        if (index >= 0 && this.RAMSections.length <= 10) {
          console.log(
            `Добавление задачи №${item.order} размером ${item.size} мб в память`
          );
          const ff = `Добавление задачи №${item.order} размером ${item.size} мб в память`;
          this.consoleMessages.push(ff);

          allFree = 0;
          const freeItems = this.RAMSections.slice(start, index + 1);
          let freeSize = 0;
          freeItems.forEach((val) => {
            freeSize = freeSize + val.size;
          });

          const busySection = {
            free: false,
            order: item.order,
            size: item.size,
          };
          const freeSection = {
            free: true,
            order: -1,
            size: freeSize - item.size,
          };
          this.RAMSections.splice(
            start,
            index - start + 1,
            busySection,
            freeSection
          );

          this.RAMSections.forEach((it) => {
            if (it.free) {
              allFree = allFree + it.size;
            }
          });
          this.sleep(5000);
          console.log(
            `Задача №${
              item.order
            } добавлена в память. Cвободной памяти осталось: ${Math.floor(
              allFree
            )} мб`
          );
          let ee = `Задача №${
            item.order
          } добавлена в память. Cвободной памяти осталось: ${Math.floor(
            allFree
          )} мб`;
          this.consoleMessages.push(ee);
        } else {
          console.log(`${MESSAGE.busy}. Ожидание завершения задачи...`);
          let ttt = `${MESSAGE.busy}. Ожидание завершения задачи...`;
          this.consoleMessages.push(ttt);
          this.sleep(5000);
          let accumulaator = allFree;
          for (let n = 0; n < this.tasksList.length; n++) {
            const currentTask = this.RAMSections.find((q) => q.order === n);
            const currentTaskIndex = this.RAMSections.findIndex(
              (q) => q.order === n
            );
            if (this.RAMSections[currentTaskIndex]) {
              this.RAMSections[currentTaskIndex].free = true;
            }
            if (currentTask && currentTask.size) {
              accumulaator = accumulaator + currentTask.size;
              console.log(
                `Задача №${currentTask.order} выполнена, свободная память - ${accumulaator} мб`
              );
              let errere = `Задача №${currentTask.order} выполнена, свободная память - ${accumulaator} мб`;
              this.consoleMessages.push(errere);
            }

            if (accumulaator >= item.size) {
              return;
            }
          }
        }
      });
    },
    random(min, max) {
      let rand = min + Math.random() * (max + 1 - min);
      return Math.floor(rand);
    },
    sleep(ms) {
      return new Promise((resolve) => setTimeout(resolve, ms));
    },
    findFree(arr, accum, taskSize) {
      const free = arr.find((item) => item.free);

      if (free && free.size) {
        accum = accum + free.size;
      } else {
        return -1;
      }

      const index = arr.indexOf(free);

      if (accum < taskSize) {
        arr[index].free = false;
        return this.findFree(arr, accum, taskSize);
      } else {
        return index;
      }
    },
  },
};
</script>

<style>
#app {
  font-family: "Avenir", Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
.os-part {
  background: green;
}
.os-part--busy {
  background: red;
}
.OS {
  width: 100%;
  height: 50px;
  background: rgba(0, 0, 0, 0.1);
  display: flex;
}
.os-part {
  height: 100%;
  background: green;
  border: 1px dashed black;
}
.os-part--busy {
  height: 100%;
  background: red;
}
</style>
