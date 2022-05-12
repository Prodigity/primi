<template>
  <v-container>
    <v-card>
      <!--<v-card-title class="text-h4">Gewicht toekennen</v-card-title>-->
      <v-simple-table>
        <thead>
          <tr>
            <th>Beoordelingspunten</th>
            <th>Hoe belangrijk is dit voor je?</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="quality in qualities" :key="quality.id">
            <td>{{ quality.name }}</td>
            <td>
              <v-card-text>
                <v-slider
                  :value="quality.bias"
                  :tick-labels="ticksLabels"
                  ticks="always"
                  tick-size="5"
                  :min="1"
                  :max="5"
                  @change="
                    $emit('biasChange', { id: quality.id, bias: $event })
                  "
                ></v-slider>
              </v-card-text>
            </td>
          </tr>
        </tbody>
      </v-simple-table>
    </v-card>
  </v-container>
</template>

<script>
export default {
  name: "BiasMatrix",

  props: {
    qualities: Array,
  },

  data: () => ({
    ticksLabels: ["Onbelangrijk", "", "", "", "Belangrijk"],
  }),
};
</script>
