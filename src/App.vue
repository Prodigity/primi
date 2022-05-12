<template>
  <v-app>
    <!-- Bar on the left -->
    <v-navigation-drawer app permanent>
      <v-list-item>
        <v-list-item-content>
          <v-list-item-title class="text-h6"> primi. </v-list-item-title>
          <v-list-item-subtitle> prioritize your life </v-list-item-subtitle>
        </v-list-item-content>
      </v-list-item>
      <v-divider></v-divider>
      <v-divider></v-divider>
      <v-list>
        <v-list-item-group v-model="selectedSubjectIndex">
          <template v-for="subject in subjects">
            <v-list-item :key="subject.id">
              <v-list-item-content>
                <v-list-item-title> {{ subject.name }} </v-list-item-title>
                <v-list-item-subtitle>
                  {{ subject.description }}
                </v-list-item-subtitle>
              </v-list-item-content>
            </v-list-item>
          </template>
        </v-list-item-group>
      </v-list>
      <v-list-item>
        <!-- Start of dialog + button -->
        <v-dialog v-model="dialog_create" persistent max-width="600px">
          <template v-slot:activator="{ on, attrs }">
            <v-btn color="primary" dark v-bind="attrs" v-on="on">
              Nieuwe item toevoegen
            </v-btn>
          </template>
          <v-card>
            <v-card-title>
              <span class="text-h5">Nieuwe item aanmaken</span>
            </v-card-title>
            <v-card-text>
              <v-container>
                <v-row>
                  <v-col cols="12">
                    <v-text-field
                      v-model="onderwerp"
                      label="Wat wil je sorteren?"
                      required
                    ></v-text-field>
                  </v-col>
                  <v-col cols="12">
                    <v-text-field
                      v-model="doel"
                      label="Met welk doel?"
                      required
                    ></v-text-field>
                  </v-col>
                </v-row>
              </v-container>
            </v-card-text>
            <v-card-actions>
              <v-spacer></v-spacer>
              <v-btn color="primary" text @click="subjectCreationCancel">
                Cancel
              </v-btn>
              <v-btn color="primary" text @click="subjectCreationOk">
                Ok
              </v-btn>
            </v-card-actions>
          </v-card>
        </v-dialog>
        <!-- End of dialog + button -->
      </v-list-item>
    </v-navigation-drawer>

    <!-- Footer, but I'm not using it right now
    <v-footer app>
      <p>footer</p>
    </v-footer>
    -->

    <!-- Start of dialog + button -->
    <v-dialog v-model="dialog_edit" persistent max-width="600px">
      <v-card>
        <v-card-title>
          <span class="text-h5">Item aanpassen</span>
        </v-card-title>
        <v-card-text>
          <v-container>
            <v-row>
              <v-col cols="12">
                <v-text-field
                  v-model="editSubject"
                  label="Wat wil je sorteren?"
                  required
                ></v-text-field>
              </v-col>
              <v-col cols="12">
                <v-text-field
                  v-model="editDescription"
                  label="Met welk doel?"
                  required
                ></v-text-field>
              </v-col>
            </v-row>
          </v-container>
        </v-card-text>
        <v-card-actions>
          <v-btn color="red" text @click="subjectEditRemove">
            Verwijderen
          </v-btn>
          <v-spacer></v-spacer>
          <v-btn color="primary" text @click="subjectEditCancel">
            Cancel
          </v-btn>
          <v-btn color="primary" text @click="subjectEditOk"> Ok </v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>

    <!-- Sizes your content based upon application components -->
    <v-main>
      <!-- Provides the application the proper gutter -->
      <v-container fluid v-if="selectedSubjectId > -1">
        <v-card class="ma-3 rounded-lg">
          <v-card-title
            >{{ selectedSubjectTitle }} <v-spacer></v-spacer>
            <v-btn icon @click="openDialogEdit">
              <v-icon>mdi-pencil</v-icon>
            </v-btn></v-card-title
          >
          <v-card-subtitle>{{ selectedSubjectDescription }} </v-card-subtitle>
          <v-toolbar>
            <v-tabs v-model="tab">
              <v-tabs-slider></v-tabs-slider>
              <v-tab> Score </v-tab>
              <v-tab> Gewicht </v-tab>
              <v-tab> Resultaat </v-tab>
            </v-tabs>
          </v-toolbar>

          <v-tabs-items v-model="tab">
            <v-tab-item>
              <score-matrix
                :subject="selectedSubject"
                :items="subjectItems()"
                :qualities="subjectQualities()"
                :scores="scores"
                @increment="increment"
                @decrement="decrement"
                @addQuality="addQuality"
                @updateQuality="updateQuality"
                @removeQuality="removeQuality"
                @addItem="addItem"
                @updateItem="updateItem"
                @removeItem="removeItem"
              >
              </score-matrix>
            </v-tab-item>
            <v-tab-item>
              <bias-matrix
                :qualities="subjectQualities()"
                @biasChange="biasChange"
              >
              </bias-matrix>
            </v-tab-item>
            <v-tab-item>
              <results-matrix
                :subject="selectedSubject"
                :items="subjectItems()"
                :qualities="subjectQualities()"
                :scores="scores"
              ></results-matrix>
            </v-tab-item>
          </v-tabs-items>
        </v-card>
      </v-container>
      <v-container v-else>
        <v-card class="ma-3 rounded-lg">
          <v-card-title>Ga aan de slag!</v-card-title>
          <v-card-text v-if="subjects.length > 0">
            Kies een item uit het linkermenu of maak een nieuwe aan
          </v-card-text>
          <v-card-text v-else>
            Maak een item aan in het linkermenu om aan de slag te gaan
          </v-card-text>
        </v-card>
      </v-container>
    </v-main>
  </v-app>
</template>

<script>
import ScoreMatrix from "./components/ScoreMatrix.vue";
import ResultsMatrix from "./components/ResultsMatrix.vue";
import BiasMatrix from "./components/BiasMatrix.vue";

export default {
  name: "App",

  components: {
    ScoreMatrix,
    ResultsMatrix,
    BiasMatrix,
  },

  mounted() {
    this.items = JSON.parse(localStorage.getItem("items")) || [];
    this.qualities = JSON.parse(localStorage.getItem("qualities")) || [];
    this.scores = JSON.parse(localStorage.getItem("scores")) || [];
    this.subjects = JSON.parse(localStorage.getItem("subjects")) || [];
  },

  methods: {
    /////////////////////
    // Syncing storage //
    /////////////////////
    updateLS_items() {
      localStorage.setItem("items", JSON.stringify(this.items));
    },
    updateLS_qualities() {
      localStorage.setItem("qualities", JSON.stringify(this.qualities));
    },
    updateLS_scores() {
      localStorage.setItem("scores", JSON.stringify(this.scores));
    },
    updateLS_subjects() {
      localStorage.setItem("subjects", JSON.stringify(this.subjects));
    },

    ////////////////////
    // Removing stuff //
    ////////////////////
    removeItem(id) {
      this.removeScoresOfItem(id);
      this.items = this.items.filter((item) => item.id != id);
      this.updateLS_items();
    },
    removeQuality(id) {
      this.removeScoresOfQuality(id);
      this.qualities = this.qualities.filter((quality) => quality.id != id);
      this.updateLS_qualities();
    },
    removeScoresOfItem(itemId) {
      this.scores = this.scores.filter((score) => itemId != score.itemId);
      this.updateLS_scores();
    },
    removeScoresOfQuality(qualityId) {
      this.scores = this.scores.filter((score) => qualityId != score.qualityId);
      this.updateLS_scores();
    },
    removeSubject(id) {
      // Get all items belonging to this subject
      const items = this.items.filter((item) => item.subjectId === id);

      // Remove all scores associated with these items
      for (const item of items) {
        this.scores = this.scores.filter((score) => item.id != score.itemId);
      }

      // Now we can remove these items
      this.items = this.items.filter((item) => item.subjectId != id);

      // Now we remove all qualities belonging to this subject
      this.qualities = this.qualities.filter(
        (quality) => quality.subjectId != id
      );

      // Finally we remove the subject itself
      this.subjects = this.subjects.filter((subject) => subject.id != id);
    },
    subjectEditRemove() {
      this.dialog_edit = false;
      const removeId = this.selectedSubjectId;
      this.selectedSubjectIndex = -1;
      this.removeSubject(removeId);
      this.updateLS_subjects();
      this.updateLS_scores();
      this.updateLS_items();
      this.updateLS_qualities();
    },

    ////////////////////
    // Updating stuff //
    ////////////////////
    updateItem({ id, newName }) {
      this.items.find((item) => item.id === id).name = newName;
      this.updateLS_items();
    },
    updateQuality({ id, newName }) {
      this.qualities.find((quality) => quality.id === id).name = newName;
      this.updateLS_qualities();
    },
    subjectEditOk() {
      this.dialog_edit = false;
      this.selectedSubject.name = this.editSubject;
      this.selectedSubject.description = this.editDescription;
      this.updateLS_subjects();
    },
    biasChange({ id, bias }) {
      this.qualities.find((quality) => quality.id === id).bias = bias;
      console.log(id, bias);
      this.updateLS_qualities();
    },
    increment(scoreId) {
      var scoreItem = this.scores.find((score) => {
        return score.id === scoreId;
      });
      if (scoreItem.score < 5) {
        scoreItem.score += 1;
      }
      this.updateLS_scores();
    },
    decrement(scoreId) {
      var scoreItem = this.scores.find((score) => {
        return score.id === scoreId;
      });
      if (scoreItem.score > 1) {
        scoreItem.score -= 1;
      }
      this.updateLS_scores();
    },

    ////////////////////
    // Creating stuff //
    ////////////////////
    subjectCreationOk() {
      this.dialog_create = false;
      this.addSubject({ name: this.onderwerp, description: this.doel });
      this.onderwerp = "";
      this.doel = "";
      this.updateLS_subjects();
    },
    addItem({ name, subjectId }) {
      // Get a unique ID for our item by incrementing the highest item ID we currently have
      var newItemId = null;
      if (this.items.length == 0) {
        newItemId = 0;
      } else {
        newItemId =
          this.items.reduce(
            (prev, current) => (prev.id > current.id ? prev : current),
            0
          ).id + 1;
      }

      // Create new item
      this.items.push({ id: newItemId, name: name, subjectId: subjectId });

      // Get an array of qualities relevant for this subject
      const subjectQualities = this.qualities.filter(
        (quality) => quality.subjectId === subjectId
      );

      // Get a unique starting ID for our item scores
      var newScoreId = null;
      if (this.scores.length == 0) {
        newScoreId = 0;
      } else {
        newScoreId =
          this.scores.reduce(
            (prev, current) => (prev.id > current.id ? prev : current),
            0
          ).id + 1;
      }

      // Give this item the default score for every quality there is on this subject
      for (const quality of subjectQualities) {
        this.scores.push({
          id: newScoreId,
          score: 3,
          itemId: newItemId,
          qualityId: quality.id,
        });
        newScoreId += 1;
      }
      this.updateLS_items();
      this.updateLS_scores();
    },
    addQuality({ name, subjectId }) {
      // Get a unique ID for our quality by incrementing the highest quality ID we currently have
      var newQualityId = null;
      if (this.qualities.length == 0) {
        newQualityId = 0;
      } else {
        newQualityId =
          this.qualities.reduce(
            (prev, current) => (prev.id > current.id ? prev : current),
            0
          ).id + 1;
      }

      // Create new quality
      this.qualities.push({
        id: newQualityId,
        name: name,
        bias: 3,
        subjectId: subjectId,
      });

      // Get an array of items relevant for this subject
      const subjectItems = this.items.filter(
        (item) => item.subjectId === subjectId
      );

      // Get a unique starting ID for our item scores
      var newScoreId = null;
      if (this.scores.length == 0) {
        newScoreId = 0;
      } else {
        newScoreId =
          this.scores.reduce(
            (prev, current) => (prev.id > current.id ? prev : current),
            0
          ).id + 1;
      }

      // Give every item that belongs to this quality the default score
      for (const item of subjectItems) {
        this.scores.push({
          id: newScoreId,
          score: 3,
          itemId: item.id,
          qualityId: newQualityId,
        });
        newScoreId += 1;
      }

      this.updateLS_qualities();
      this.updateLS_scores();
    },
    addSubject({ name, description }) {
      // Get new unique ID
      var newSubjectId = null;
      if (this.subjects.length == 0) {
        newSubjectId = 0;
      } else {
        newSubjectId =
          this.subjects.reduce(
            (prev, current) => (prev.id > current.id ? prev : current),
            0
          ).id + 1;
      }

      // Create new subject
      this.subjects.push({
        id: newSubjectId,
        name: name,
        description: description,
      });
      this.updateLS_subjects();
    },

    openDialogEdit() {
      this.dialog_edit = true;
      this.editSubject = this.selectedSubjectTitle;
      this.editDescription = this.selectedSubjectDescription;
    },
    subjectEditCancel() {
      this.dialog_edit = false;
    },
    subjectCreationCancel() {
      this.dialog_create = false;
      this.onderwerp = "";
      this.doel = "";
    },

    subjectItems() {
      return this.items.filter(
        (item) => item.subjectId === this.selectedSubjectId
      );
    },
    subjectQualities() {
      return this.qualities.filter(
        (quality) => quality.subjectId === this.selectedSubjectId
      );
    },
    isNotLast({ list, index }) {
      return index < list.length - 1;
    },
  },

  computed: {
    selectedSubjectId() {
      console.log(this.selectedSubjectIndex);
      if (
        this.selectedSubjectIndex == null ||
        this.selectedSubjectIndex == -1
      ) {
        return -1;
      } else {
        return this.subjects[this.selectedSubjectIndex].id;
      }
    },
    selectedSubject() {
      return this.subjects.find(
        (subject) => subject.id === this.selectedSubjectId
      );
    },
    selectedSubjectTitle() {
      return this.subjects.find(
        (subject) => subject.id === this.selectedSubjectId
      ).name;
    },
    selectedSubjectDescription() {
      return this.subjects.find(
        (subject) => subject.id === this.selectedSubjectId
      ).description;
    },
    selectedSubjectItems() {
      return this.items.filter(
        (item) => item.subjectId == this.selectedSubject
      );
    },
    selectedSubjectQualities() {
      return this.qualities.filter(
        (item) => item.subjectId == this.selectedSubject
      );
    },
  },

  data: () => ({
    e1: 1,
    selectedSubjectIndex: -1,

    tab: null,

    dialog_create: false,
    onderwerp: "",
    doel: "",

    dialog_edit: false,
    editSubject: "",
    editDescription: "",

    subjects: [],

    items: [],

    qualities: [],

    scores: [],
  }),
};
</script>

<style>
.tiny-btn {
  min-width: 0 !important;
  padding: 0 !important;
}
tbody tr:nth-of-type(odd) {
  background-color: rgba(0, 0, 0, 0.1);
}
tbody td:nth-of-type(odd) {
  background-color: rgba(0, 0, 0, 0.1);
}
thead th:nth-of-type(odd) {
  background-color: rgba(0, 0, 0, 0.1);
}
.appheader {
  background-color: rgba(255, 200, 0, 0.5);
}
.v-list {
  padding: 0 !important;
}
</style>
