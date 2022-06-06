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
          <button class="btn btn-secondary" @click="replace">Replace</button>
          <button @click="reverse">Reverse</button>
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
        :list="list"
        :disabled="!enabled"
        item-key="name"
        class="list-group"
        ghost-class="ghost"
        :move="checkMove"
        @start="dragging = true"
        @end="dragging = false"
        handle=".handle-only-this"
        animation="200"
      >
        <template #header>
          <input type="time" v-model="firstElement.content" />
        </template>
        <template #item="{ element }">
          <div v-if="element.id !== 0 && element.id !== lastElement.id">
            <textarea
              v-if="element.draggable"
              v-model="element.content"
              placeholder="Write your action"
            ></textarea>
            <input
              type="time"
              v-if="!element.draggable"
              v-model="element.content"
            />
            <span v-if="element.draggable" class="handle-only-this">D</span>
          </div>
        </template>
        <template #footer>
          <input type="time" v-model="lastElement.content" />
        </template>
      </draggable>
    </div>

    <rawDisplayer class="col-3" :value="list" title="List" />
  </div>
</template>

<script>
/* eslint-disable no-console */
import draggable from "vuedraggable";

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
      list: [
        { id: 0, content: "09:15", draggable: false },
        { id: 1, content: "", draggable: true },
        { id: 2, content: "17:45", draggable: false },
      ],
      dragging: false,
    };
  },
  computed: {
    draggingInfo() {
      return this.dragging ? "under drag" : "";
    },
    firstElement() {
      return this.list[0];
    },
    lastElement() {
      return this.list[this.list.length - 1];
    },
  },
  methods: {
    add: function () {
      const lastID = this.list.length - 1;
      const lastTime = this.list[lastID].content;
      this.list.push({ id: lastID + 1, content: "", draggable: true });
      this.list.push({ id: lastID + 2, content: lastTime, draggable: false });
    },
    replace: function () {
      this.list = [{ name: "Edgard", id: id++ }];
    },
    checkMove: function (e) {
      window.console.log("Future index: " + e.draggedContext.futureIndex);
    },
    reverse: function () {
      this.list.reverse();
    },
  },
};
</script>