<template>
  <v-container>
    <v-card>
      <!--<v-card-title class="text-h4">Score tabel</v-card-title>-->
      <v-simple-table>
        <thead>
          <tr>
            <th>{{ subject.name }}</th>
            <th
              class="text-center"
              v-for="quality in qualities"
              :key="quality.id"
            >
              <v-text-field
                :value="quality.name"
                @change="editQuality({ id: quality.id, newName: $event })"
              >
              </v-text-field>
            </th>
            <th>
              <v-text-field
                v-model="qualityname"
                @keydown.enter.prevent="addQuality"
                label="Voeg beoordelingspunten toe"
              ></v-text-field>
            </th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="item in items" :key="item.id">
            <td>
              <v-text-field
                :value="item.name"
                @change="editItem({ id: item.id, newName: $event })"
              >
              </v-text-field>
            </td>
            <td
              class="text-center"
              v-for="score in itemScores(item.id)"
              :key="score.id"
            >
              <v-btn
                class="tiny-btn"
                plain
                :color="minusButtonColorAtScore(score.score)"
                @click="$emit('decrement', score.id)"
              >
                <span class="text-h4" style="text-align: center">&lt;</span>
              </v-btn>
              <span class="text-h4">{{ scoreToEmoji(score.score) }}</span>
              <v-btn
                class="tiny-btn"
                plain
                :color="plusButtonColorAtScore(score.score)"
                @click="$emit('increment', score.id)"
              >
                <span class="text-h4">></span>
              </v-btn>
            </td>
            <td></td>
          </tr>
          <tr>
            <td>
              <v-text-field
                v-model="itemname"
                @keydown.enter.prevent="addItem"
                :label="'Voeg ' + subject.name.toLowerCase() + ' toe'"
              ></v-text-field>
            </td>
            <td
              class="text-center"
              v-for="quality in qualities"
              :key="quality.id"
            ></td>
            <td></td>
          </tr>
        </tbody>
      </v-simple-table>
    </v-card>
  </v-container>
</template>

<script>
export default {
  name: "ScoreMatrix",

  props: {
    subject: Object,
    items: Array,
    qualities: Array,
    scores: Array,
  },

  data: () => ({
    itemname: "",
    qualityname: "",
  }),

  methods: {
    editItem({ id, newName }) {
      if (newName == "") {
        this.$emit("removeItem", id);
      } else {
        this.$emit("updateItem", { id, newName });
      }
    },
    editQuality({ id, newName }) {
      if (newName == "") {
        this.$emit("removeQuality", id);
      } else {
        this.$emit("updateQuality", { id, newName });
      }
    },
    log(thing) {
      console.log(thing);
    },
    scoreToEmoji(score) {
      if (score == 1) {
        return "😭";
      } else if (score == 2) {
        return "😒";
      } else if (score == 3) {
        return "😐";
      } else if (score == 4) {
        return "😁";
      } else if (score == 5) {
        return "😍";
      }
    },

    addItem() {
      this.$emit("addItem", {
        name: this.itemname,
        subjectId: this.subject.id,
      });
      this.itemname = "";
    },
    addQuality() {
      this.$emit("addQuality", {
        name: this.qualityname,
        subjectId: this.subject.id,
      });
      this.qualityname = "";
    },
    itemScores(itemId) {
      return this.scores.filter((score) => score.itemId == itemId);
    },
    minusButtonColorAtScore(score) {
      if (score > 1) {
        return "primary";
      } else {
        return "secondary";
      }
    },
    plusButtonColorAtScore(score) {
      if (score < 5) {
        return "primary";
      } else {
        return "secondary";
      }
    },
  },
};
</script>
