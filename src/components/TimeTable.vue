<template>
  <div>
    <draggable
      :list="emptyListOfDeleteArea"
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

    <div v-for="day in timeTable" :key="day.id">
      {{ day.date }}
      <draggable
        :list="day.rows"
        :disabled="!enabled"
        item-key="id"
        @change="modifyTimeWhenMoved"
        @start="enableDeleteArea"
        @end="
          disableDeleteArea();
          clearEmptyListOfDeleteArea();
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
              v-model="element.action"
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
    <button class="btn btn-secondary" @click="copy">Copy</button>
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

function getConvertedTimeTable(timeTable, lastTime) {
  var result = [];
  //1日以上日付をまたいだ時の処理を考える。
  for (var i = 0; i < timeTable.length; i++) {
    var isAddedDay = false; //dayを書くかどうかのフラグ
    //rowに無いactionをresultに追加するための処理
    //前の日付が存在する場合
    if (0 < i) {
      //丸一日以上日付をまたいだ場合
      //timeTableにある、ひとつ前の日付
      var formerDate = new Date(
        timeTable[i - 1].date.substr(0, 4),
        timeTable[i - 1].date.substr(5, 2),
        timeTable[i - 1].date.substr(8, 2)
      );
      //それを1日後にする。
      formerDate.setDate(formerDate.getDate() + 1);
      //for文で回している最中の日付
      var thisDate = new Date(
        timeTable[i].date.substr(0, 4),
        timeTable[i].date.substr(5, 2),
        timeTable[i].date.substr(8, 2)
      );
      //for文で回している最中の日付の前日まで
      while (formerDate < thisDate) {
        //dayは、日付にする。
        //timeを、00:00~24:00にする。
        const year = ("0000" + formerDate.getFullYear()).slice(-4);
        const month = ("00" + formerDate.getMonth()).slice(-2);
        const date = ("00" + formerDate.getDate()).slice(-2);
        result.push({
          date: `${year}-${month}-${date}`,
          time: "00:00~24:00",
          action:
            timeTable[i - 1].rows[timeTable[i - 1].rows.length - 1].action,
        });
        //1日加算する。
        formerDate.setDate(formerDate.getDate() + 1);
      }

      //rowが1つ以上で、最初のtimeが00:00でないとき
      if (
        timeTable[i].rows.length > 0 &&
        timeTable[i].rows[0].time !== "00:00"
      ) {
        //timeを、00:00~最初のtimeにする。
        result.push({
          date: timeTable[i].date,
          time: `00:00~${timeTable[i].rows[0].time}`,
          action:
            timeTable[i - 1].rows[timeTable[i - 1].rows.length - 1].action,
        });
        isAddedDay = true;
      }

      //rowが0個で、lastTimeが00:00ではないとき
      if (timeTable[i].rows.length == 0 && lastTime !== "00:00") {
        //timeを、00:00~lastTimeにする。
        result.push({
          date: timeTable[i].date,
          time: `00:00~${lastTime}`,
          action:
            timeTable[i - 1].rows[timeTable[i - 1].rows.length - 1].action,
        });
        isAddedDay = true;
      }
    }

    //rowに存在するactionをresultに追加するための処理
    for (var j = 0; j < timeTable[i].rows.length; j++) {
      //日付を記述するかどうかを判断する処理
      var date = "";
      if (!isAddedDay) {
        date = timeTable[i].date;
        isAddedDay = true;
      }

      //次のrowがあるとき
      if (j < timeTable[i].rows.length - 1) {
        //今のrowの時間~次のrowの時間を、timeとする。
        result.push({
          date: date,
          time: `${timeTable[i].rows[j].time}~${timeTable[i].rows[j + 1].time}`,
          action: timeTable[i].rows[j].action,
        });
        continue;
      }

      //次の日付があるとき
      if (i < timeTable.length - 1) {
        //今のrowの時間~24:00を、timeとする。
        result.push({
          date: date,
          time: `${timeTable[i].rows[j].time}~24:00`,
          action: timeTable[i].rows[j].action,
        });
        continue;
      }

      //最後の日付で、最後のrowのとき
      result.push({
        date: date,
        time: `${timeTable[i].rows[j].time}~${lastTime}`,
        action: timeTable[i].rows[j].action,
      });
    }
  }
  return result;
}

function getMDTimeTable(convertedTimeTable) {
  var result = "";
  result += "|日付|時間|行動|\n";
  result += "|:-:|:-:|:-:|\n";
  for (const row of convertedTimeTable) {
    result += `|${row.date}|${row.time}|${row.action}|\n`;
  }
  return result;
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
        {
          id: 0,
          date: "2023-02-01",
          rows: [
            {
              id: 0,
              time: "09:15",
              action: "email checkig",
            },
            {
              id: 1,
              time: "10:00",
              action: "coding",
            },
            {
              id: 2,
              time: "12:00",
              action: "lunch",
            },
          ],
        },
        {
          id: 1,
          date: "2023-02-02",
          rows: [
            {
              id: 3,
              time: "09:15",
              action: "email checkig",
            },
            {
              id: 4,
              time: "10:00",
              action: "coding",
            },
            {
              id: 5,
              time: "12:00",
              action: "lunch",
            },
          ],
        },
      ],
      lastTime: "18:45",
      emptyListOfDeleteArea: [],
    };
  },
  computed: {
    draggingInfo() {
      return this.dragging ? "under drag" : "";
    },
    getNull2() {
      //ボツ
      return null;
      //var result = "";

      //return result;
    },
    getNull() {
      //ボツ
      return null;
      /**
       * jの判定について共通化できる部分があるのでコーディングの際に修正をする。
       * result=[]
       * for 日付(i)
       *    for タスク(j)
       *        if 同じ日付の中で次のタスクがある場合
       *            (pushする要素は1つのみ。)
       *            const day = (j==0) ? days[i] : ""
       *            result.push({day:day,time:times[j]~times[j+1],action:actions[j]})
       *        if 次の日付が無い場合(1つ目のif文を満たしていないので、同じ日付の中で一番最後のタスク、という前提がある。)
       *            (pushする要素は1つのみ。)
       *            const day = (j==0) ? days[i] : ""
       *            result.push({day:day,time:times[j]~lastTime,action:actions[j]})
       *        if  次の日付が有り、かつその要素が0個の場合(1つ目のif文を満たしていないので、同じ日付の中で一番最後のタスク、という前提がある。)
       *            (pushする要素は1つ以上。)
       *            const day = (j==0) ? days[i] : ""
       *            result.push({day:day,time:times[j]~24:00,action:actions[j]})
       *            for その日付から、次の日付の前日まで(k)
       *                result.push({day:dayから(k+1)日後,time:00:00~24:00,action:actions[j]})
       *            if lastTime == "00:00"
       *                (pushする要素は0個。)
       *            else
       *                (pushする要素は1つのみ。)
       *                result.push({day:days[i+1],time:00:00~lasTime,action:actions[j]})
       *        (その他の場合)(その日の最後の要素で、次の日付があり、かつその要素が1個以上の場合)
       *        const day = (j==0) ? days[i] : ""
       *        result.push({day:day,time:times[j]~24:00,action:actions[j]})
       *        result.push({days[i+1],time:00:00~days[i].times[0],action:actions[j]})
       *
       * 次の日に00:00のみのタスクとlastTimeが00:00だとバグるかもしれない。
       * あと、前の日付があった場合は、その日の最初の要素でもdays[i]とはならない。これは、j==0のときに、さらにその日の最初の要素が00:00から始まっていればdayをdays[i]にして、それ以外は""とすることで解決する。
       */
    },
  },
  methods: {
    add: function () {
      const newRow = { id: this.lastID + 1, time: this.lastTime, action: "" };
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
    clearEmptyListOfDeleteArea: function () {
      this.emptyListOfDeleteArea = [];
    },
    copy: function () {
      const converted = getConvertedTimeTable(this.timeTable, this.lastTime);
      const md = getMDTimeTable(converted);
      navigator.clipboard.writeText(md);
      alert("コピーしました");
    },
  },
};
</script>