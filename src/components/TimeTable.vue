<template>
  <div class="row">
    <div class="col-2">
      <div class="form-group">
        <div
          class="btn-group-vertical buttons"
          role="group"
          aria-label="Basic example"
        >
          <button class="btn btn-secondary" @click="add">Add</button>
        </div>

        <div class="form-check">
          <input
            id="disabled"
            type="checkbox"
            v-model="enabled"
            class="form-check-input"
          />
          <label class="form-check-label" for="disabled">DnD enabled</label>
        </div>
      </div>
    </div>

    <div class="col-6">
      <h3>Draggable {{ draggingInfo }}</h3>

      <draggable
        :list="timeTable"
        :disabled="!enabled"
        item-key="name"
        class="list-group"
        ghost-class="ghost"
        :move="checkMove"
        @start="dragging = true"
        @end="dragging = false"
        @change="modifyTimeWhenMoved"
        handle=".handle-only-this"
        animation="350"
      >
        <template #item="{ element }">
          <div>
            <input type="time" v-model="element.time" />
            <br />
            <textarea
              v-model="element.text"
              placeholder="Write your action"
            ></textarea>
            <span class="handle-only-this">Drag</span>
          </div>
        </template>
        <template #footer>
          <input type="time" v-model="lastTime" />
        </template>
      </draggable>
    </div>

    <rawDisplayer class="col-3" :value="list" title="List" />
  </div>
</template>

<script>
/* eslint-disable no-console */
import draggable from "vuedraggable";

function isASameOrBeforeB(a, b) {
  const hourA = Number(a.substr(0, 2));
  const hourB = Number(b.substr(0, 2));
  if (hourA < hourB) return true;
  if (hourB < hourA) return false;

  const minutesA = Number(a.substr(3, 2));
  const minutesB = Number(b.substr(3, 2));
  if (minutesA <= minutesB) return true;
  return false;
}

let id = 1;
export default {
  name: "TimeTable",
  display: "TimeTable",
  order: 0,
  components: {
    draggable,
  },
  data() {
    return {
      enabled: true,
      dragging: false,
      timeTable: [
        { id: 0, time: "09:15", text: "meeting" },
        { id: 1, time: "12:00", text: "lunch" },
        { id: 2, time: "13:00", text: "coding" },
        { id: 3, time: "15:00", text: "meeting" },
      ],
      lastTime: "18:45",
    };
  },
  computed: {
    draggingInfo() {
      return this.dragging ? "under drag" : "";
    },
    lastID() {
      var maxID = 0;
      for (const row of this.timeTable) {
        if (maxID < row.id) {
          maxID = row.id;
        }
      }
      return maxID;
    },
  },
  methods: {
    add: function () {
      const newRow = { id: this.lastID + 1, time: this.lastTime, text: "" };
      this.timeTable.push(newRow);
    },
    checkMove: function (e) {
      //window.console.log("Future index: " + e.draggedContext.futureIndex);
    },
    a: function (message) {
      //function for debugging
      alert(message);
    },
    ai: function (item) {
      //function for debugging
      alert(item.moved.newIndex);
    },
    modifyTimeWhenMoved: function (item) {
      const newIndex = item.moved.newIndex;

      //動かした要素
      const movedElement = this.timeTable[newIndex];

      var timeAbove = null;
      if (newIndex === 0) {
        timeAbove = movedElement.time;
      } else {
        timeAbove = this.timeTable[newIndex - 1].time;
      }

      var timeBelow = null;
      if (newIndex === this.timeTable.length - 1) {
        timeBelow = this.lastTime;
      } else {
        timeBelow = this.timeTable[newIndex + 1].time;
      }

      if (
        !isASameOrBeforeB(timeAbove, movedElement.time) ||
        !isASameOrBeforeB(movedElement.time, timeBelow)
      ) {
        this.timeTable[newIndex].time = timeBelow;
      }
    },
    modifyTimeWhenTimeChanged(){
      alert("wrong time input")
    }
  },
};
</script>