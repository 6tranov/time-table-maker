<template>
  <div>
    <draggable
      :list="emptyListOfDeleteArea"
      :disabled="!enabled"
      item-key="id"
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
        @change="modifyTimeWhenMoved($event, getDateIndexFromDateID(day.id))"
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
            <input
              type="time"
              v-model="element.time"
              @focus="setOldTime($event.target.value)"
              @change="
                modifyTimeWhenChanged(
                  getDateIndexAndRowIndexPairFromRowID(element.id)
                )
              "
              @keydown.enter="focusNextAction(element.id)"
              :ref="'time' + element.id"
            />
            <br />
            <input
              type="text"
              v-model="element.action"
              placeholder="Write your action"
              @keydown.enter="
                focusNextTime(getDateIndexAndRowIndexPairFromRowID(element.id))
              "
              :ref="'action' + element.id"
            />
            <span class="handle-only-this">Drag</span>
          </div>
        </template>
      </draggable>
    </div>
    <div>
      <input type="time" v-model="lastTime" ref="lastTime" />
    </div>

    <button class="btn btn-secondary" @click="add">Add</button>
    <button class="btn btn-secondary" @click="copyToClipboard">Copy</button>
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

function timeRef(timeID) {
  return `time${timeID}`;
}
function actionRef(actionID) {
  return `action${actionID}`;
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
      lastTimeRef: "lastTime",
      emptyListOfDeleteArea: [],
      oldTime: "00:00",
    };
  },
  computed: {
    draggingInfo() {
      return this.dragging ? "under drag" : "";
    },
  },
  methods: {
    //dateIDから、dateIndexを取得する。
    getDateIndexFromDateID(dateID) {
      for (var index = 0; index < this.timeTable.length; index++) {
        if (this.timeTable[index].id === dateID) {
          return index;
        }
      }
      return null;
    },
    //rowのidから、(dateIndex,rowIndex)を取得する。
    getDateIndexAndRowIndexPairFromRowID(rowID) {
      for (var dateIndex = 0; dateIndex < this.timeTable.length; dateIndex++) {
        for (
          var rowIndex = 0;
          rowIndex < this.timeTable[dateIndex].rows.length;
          rowIndex++
        ) {
          if (this.timeTable[dateIndex].rows[rowIndex].id === rowID) {
            return [dateIndex, rowIndex];
          }
        }
      }
      return null;
    },
    //新しいrowを追加する。
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
    //rowの削除エリアを無効化する。
    disableDeleteArea: function () {
      document.getElementById("deleteArea").style.backgroundColor = "white";
    },
    clearEmptyListOfDeleteArea: function () {
      this.emptyListOfDeleteArea = [];
    },
    //作成したtimeTableを、マークダウン形式でクリップボードにコピーする。
    copyToClipboard: function () {
      const converted = getConvertedTimeTable(this.timeTable, this.lastTime);
      const md = getMDTimeTable(converted);
      navigator.clipboard.writeText(md);
      alert("copied.");
    },
    //rowを移動したとき、時間の整合性が取れるようにする。
    modifyTimeWhenMoved(event, dateIndex) {
      //rowの移動元の配列に対しては、何も行わない。
      if (typeof event.removed !== "undefined") {
        return;
      }

      //rowの移動先の配列に対して、timeの順番の整合性を取る。
      //dragが行われた後の、dateとrowのIndexを取得する。
      const newDateIndex = dateIndex;
      var newRowIndex = null;
      //同じ日付の中でrowの交換が行われた場合
      if (typeof event.moved !== "undefined") {
        newRowIndex = event.moved.newIndex;
      }
      //日付間でrowの交換が行われた場合
      if (typeof event.added !== "undefined") {
        newRowIndex = event.added.newIndex;
      }

      //dragされたrowのtimeを、下のtimeに合わせる。
      this.setSameTimeAsBelow(newDateIndex, newRowIndex);
    },
    //dateとrowのindexを指定して、そのrowのtimeの整合性を取る。
    setSameTimeAsBelow(dateIndex, rowIndex) {
      //1つ下にrowがあるとき、そのtimeにする。
      if (rowIndex < this.timeTable[dateIndex].rows.length - 1) {
        this.timeTable[dateIndex].rows[rowIndex].time =
          this.timeTable[dateIndex].rows[rowIndex + 1].time;
        return;
      }

      //(1番下のrowという前提のもと)1番最後の日付のとき、lastTimeにする。
      if (dateIndex == this.timeTable.length - 1) {
        this.timeTable[dateIndex].rows[rowIndex].time = this.lastTime;
        return;
      }

      //(1番下の要素で次の日付があるという前提のもと)上のrowがあるとき、そのtimeにする。
      if (0 < rowIndex) {
        this.timeTable[dateIndex].rows[rowIndex].time =
          this.timeTable[dateIndex].rows[rowIndex - 1].time;
        return;
      }

      //それ以外の(新しい日付にドラッグした)とき、00:00にする。
      this.timeTable[dateIndex].rows[rowIndex].time = "00:00";
      return;
    },
    setOldTime(oldTime) {
      this.oldTime = oldTime;
    },
    //timeを編集したときに、順序の整合性を取る。
    modifyTimeWhenChanged(dateIndexAndRowIndexPair) {
      const dateIndex = dateIndexAndRowIndexPair[0];
      const rowIndex = dateIndexAndRowIndexPair[1];
      //誤った順序のとき
      if (!this.isCorrectTimeOrder(dateIndex, rowIndex)) {
        //timeを、編集前のtimeに設定する。
        this.timeTable[dateIndex].rows[rowIndex].time = this.oldTime;
        //順序が正しくないことを示すアラートを表示する。
        alert("wrong time order.");
        //
        const rowID = this.timeTable[dateIndex].rows[rowIndex].id;
        this.$refs[`time${rowID}`][0].focus();
      }
    },
    isCorrectTimeOrder(dateIndex, rowIndex) {
      var formerTime = null;
      var thisTime = this.timeTable[dateIndex].rows[rowIndex].time;
      var laterTime = null;

      //formerDateの設定
      //2番目以降のrow
      if (0 < rowIndex) {
        //1つ前のtimeをformerDateとして設定する
        formerTime = this.timeTable[dateIndex].rows[rowIndex - 1].time;

        //最初のrowのとき
      } else {
        //00:00に設定する
        formerTime = "00:00";
      }

      //laterDateの設定
      //最後のrowではないとき
      if (rowIndex < this.timeTable[dateIndex].rows.length - 1) {
        //次のrowのtimeを設定する
        laterTime = this.timeTable[dateIndex].rows[rowIndex + 1].time;

        //(最後のrowで)最後の日付のとき
      } else if (dateIndex === this.timeTable.length - 1) {
        //lastTimeを設定する
        laterTime = this.lastTime;

        //(最後のrowで)次の日付があるとき
      } else {
        //23:59を設定する
        laterTime = "23:59";
      }

      //順序の整合性を確認する。
      return (
        isASameOrBeforeB(formerTime, thisTime) &&
        isASameOrBeforeB(thisTime, laterTime)
      );
    },
    focusNextAction(timeID) {
      /**
       * timeの次にあるactionはidが同じなので、
       * 単純にaction${ID}と指定すればよい。
       * [0]としているのは、原因はよくわからないが
       * this.$refs[ref]が配列として扱われており、
       * その1つ目の要素を取得すればいいかららしい。
       */
      this.$refs[`action${timeID}`][0].focus();
    },
    focusNextTime(dateIndexAndRowIndexPair) {
      const dateIndex = dateIndexAndRowIndexPair[0];
      const rowIndex = dateIndexAndRowIndexPair[1];
      //dateIndexとrowIndexから、次のtimeのrefを特定する。
      var nextTimeRef = null;

      //次のrowがある場合
      if (rowIndex < this.timeTable[dateIndex].rows.length - 1) {
        //次の要素のidから、refがわかる。
        const id = this.timeTable[dateIndex].rows[rowIndex + 1].id;
        nextTimeRef = timeRef(id);

        //(最後のrowという前提のもと)次の日付がある場合
      } else if (dateIndex < this.timeTable.length - 1) {
        //次の日付が空の場合
        if (this.timeTable[dateIndex + 1].rows.length === 0) {
          //lastTimeをrefにする。
          nextTimeRef = this.lastTimeRef;

          //次の日付が空ではない場合
        } else {
          //次の日の最初のrowをrefにする。
          const id = this.timeTable[dateIndex + 1].rows[0].id;
          nextTimeRef = timeRef(id);
        }
        //(最後のrowという前提のもと)次の日付が無い場合
      } else {
        //lastTimeをrefにする。
        nextTimeRef = this.lastTimeRef;
      }

      //refにfocusを当てる。
      /**
       * 原因は良くわからないが、
       * lastTimeかどうかで、
       * [0]をつけるかどうかが決まる。
       */
      //lastTimeをfocusするとき
      if (nextTimeRef === this.lastTimeRef) {
        this.$refs[nextTimeRef].focus();

        //lastTime以外をfocusするとき
      } else {
        this.$refs[nextTimeRef][0].focus();
      }
    },
  },
};
</script>