<template>
  <v-container>
    <v-card>
      <!--<v-card-title class="text-h4">Resultaat</v-card-title>-->
      <v-simple-table>
        <thead>
          <tr>
            <th>{{ subject.name }}</th>
            <th>Score</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="item in itemScores" :key="item.id">
            <td v-if="isUrl(item.name)">
              <a target="_blank" :href="item.name">{{ item.name }}</a>
            </td>
            <td v-else>{{ item.name }}</td>
            <td>{{ item.score }}%</td>
          </tr>
        </tbody>
      </v-simple-table>
    </v-card>
  </v-container>
</template>

<script>
export default {
  name: "ResultsMatrix",

  props: {
    subject: Object,
    items: Array,
    qualities: Array,
    scores: Array,
  },

  methods: {
    isUrl(str) {
      let url;
      try {
        url = new URL(str);
      } catch (_) {
        return false;
      }
      return url.protocol === "http:" || url.protocol === "https:";
    },
  },

  computed: {
    maxScore() {
      var maxscore = 0;
      for (const quality of this.qualities) {
        maxscore += quality.bias * 5;
      }
      return maxscore;
    },
    itemScores() {
      var itemscores = [];
      for (const item of this.items) {
        const scores = this.scores.filter((score) => score.itemId === item.id);
        const me = this;
        var total = scores.reduce(function (a, b) {
          const quality = me.qualities.find(
            (quality) => b.qualityId === quality.id
          );
          return a + b.score * quality.bias;
        }, 0);
        total = (total / this.maxScore) * 100;
        itemscores.push({
          id: item.id,
          name: item.name,
          score: total.toFixed(2),
        });
      }
      itemscores.sort((a, b) => b.score - a.score);
      return itemscores;
    },
  },
};
</script>
