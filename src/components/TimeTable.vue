<template>
  <draggable
    :list="emptyList"
    :disabled="!enabled"
    item-key="id"
    @change="modifyTimeWhenMoved"
    handle=".handle-only-this"
    animation="350"
    group="day"
    style="border: solid; height: 57px"
    id="deleteArea"
  >
    <template #item="{ element }">
      <div>
        {{ element }}
      </div>
    </template>
  </draggable>

  <div v-for="day in newTimeTable" :key="day.id">
    {{ day.date }}
    <draggable
      :list="day.contents"
      :disabled="!enabled"
      item-key="id"
      @change="modifyTimeWhenMoved"
      @start="enableDeleteArea"
      @end="
        disableDeleteArea();
        clearEmptyList();
      "
      handle=".handle-only-this"
      animation="350"
      group="day"
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
    </draggable>
  </div>
  <div>
    <input type="time" v-model="lastTime" />
  </div>

  <button class="btn btn-secondary" @click="add">Add</button>
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
      newTimeTable: [
        {
          id: 0,
          date: "2023-02-01",
          contents: [
            {
              id: 0,
              time: "09:15",
              text: "email checkig",
            },
            {
              id: 1,
              time: "10:00",
              text: "coding",
            },
            {
              id: 2,
              time: "12:00",
              text: "lunch",
            },
          ],
        },
        {
          id: 1,
          date: "2023-02-02",
          contents: [
            {
              id: 3,
              time: "09:15",
              text: "email checkig",
            },
            {
              id: 4,
              time: "10:00",
              text: "coding",
            },
            {
              id: 5,
              time: "12:00",
              text: "lunch",
            },
          ],
        },
      ],
      lastTime: "18:45",
      emptyList: [],
    };
  },
  computed: {
    draggingInfo() {
      return this.dragging ? "under drag" : "";
    },
  },
  methods: {
    add: function () {
      const newRow = { id: this.lastID + 1, time: this.lastTime, text: "" };
      this.timeTable.push(newRow);
    },
    checkMove: function (e) {
      //alert("Future index: " + e.draggedContext.futureIndex);
      //document.body.style.backgroundColor = "red";
      document.getElementById("deleteArea").style.backgroundColor = "red";
    },
    a: function (message) {
      //function for debugging
      alert(message);
    },
    ai: function (item) {
      //function for debugging
      alert(item.moved.newIndex);
    },
    enableDeleteArea: function () {
      document.getElementById("deleteArea").style.backgroundColor = "red";
    },
    disableDeleteArea: function () {
      document.getElementById("deleteArea").style.backgroundColor = "white";
    },
    clearEmptyList: function () {
      this.emptyList = [];
    },
  },
};
</script>