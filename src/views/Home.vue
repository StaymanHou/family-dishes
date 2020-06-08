<template>
  <div class="page-container">
    <md-app md-mode="reveal">
      <md-app-toolbar class="md-primary">
        <h3 class="md-title" style="flex: 1">我家的家常菜</h3>
        <md-button class="md-icon-button" @click="scrollToTop">
          <md-icon>vertical_align_top</md-icon>
        </md-button>
      </md-app-toolbar>

      <md-app-content>
        <md-card>
          <md-card-content>
            <md-autocomplete
              class="search"
              v-model="searchTextHolder"
              :md-options="[]"
              @md-changed="searchChanged"
              md-layout="box">
              <label>Search...</label>
            </md-autocomplete>
            <md-divider></md-divider>
            <md-list v-for="(value, key) in tagTypes" class="md-line" :key="key">
              <md-subheader>{{ value }}</md-subheader>
              <md-list-item class="horizontal-scroll">
                <div>
                  <md-chip class="md-primary" v-for="tag in selectedTags[key]" :key="tag" md-clickable @click="deselectTag(key, tag)">{{ tag }}</md-chip>
                  <md-chip v-for="tag in tags[key]" :key="tag" v-if="!selectedTags[key].includes(tag) && (searchTerm ? tag.includes(searchTerm) : true)" md-clickable @click="selectTag(key, tag)">{{ tag }}</md-chip>
                </div>
              </md-list-item>
            </md-list>
            <md-list class="md-line">
              <md-subheader>耗时</md-subheader>
              <md-list-item class="horizontal-scroll">
                <div>
                  <md-chip class="md-primary" v-for="preparation_time in selectedPreparationTimes" :key="preparation_time" md-clickable @click="deselectTag('preparation_times', preparation_time)">{{ preparation_time }}</md-chip>
                  <md-chip v-for="preparation_time in preparation_times" :key="preparation_time" v-if="!selectedPreparationTimes.includes(preparation_time)" md-clickable @click="selectTag('preparation_times', preparation_time)">{{ preparation_time }}</md-chip>
                </div>
              </md-list-item>
            </md-list>
          </md-card-content>
        </md-card>

        <div style="height: 0; overflow-y: visible;">
          <md-progress-bar md-mode="indeterminate" v-if="loading"></md-progress-bar>
        </div>

        <div style="height: 12px;"></div>

        <div class="md-layout md-alignment-top-space-around">
          <md-card v-for="ranking in rankings.slice(0,this.showCount)" :key="ranking.index" class="md-layout-item md-xlarge-size-20 md-large-size-25 md-medium-size-40 md-small-size-80 md-xsmall-size-90">
            <md-card-header>
              <div class="md-title text-align-left">{{ recipes[ranking.index].title }}</div>
            </md-card-header>

            <md-card-content>
              <div class="text-align-left">
                <md-icon class="yellow" v-for="n in parseInt(recipes[ranking.index].rating/2)">star</md-icon>
                <md-icon class="yellow" v-for="n in recipes[ranking.index].rating%2">star_half</md-icon>
                <md-icon v-for="n in parseInt((10-recipes[ranking.index].rating)/2)">star_border</md-icon>
              </div>
              <div class="child-top-margin">
                <md-chip class="md-primary" v-if="isInSelectedTimeRange(recipes[ranking.index].preparation_time)">{{ recipes[ranking.index].preparation_time }} min</md-chip>
                <md-chip class="md-primary" v-for="tag in selectedTagsMerged" :key="tag" v-if="recipes[ranking.index].tags.includes(tag)">{{ tag }}</md-chip>
                <md-chip v-if="!isInSelectedTimeRange(recipes[ranking.index].preparation_time)">{{ recipes[ranking.index].preparation_time }} min</md-chip>
                <md-chip v-for="tag in recipes[ranking.index].tags" :key="tag" v-if="!selectedTagsMerged.includes(tag)">{{ tag }}</md-chip>
              </div>
            </md-card-content>

            <md-card-actions>
              <md-button class="md-primary" @click="openInNew(recipes[ranking.index].url)">
                <md-icon>open_in_new</md-icon>
              </md-button>
            </md-card-actions>
          </md-card>
        </div>

        <div style="height: 12px;"></div>

        <div>
          <md-button class="md-raised" @click="loadMore()" v-if="showCount < recipes.length">Load More</md-button>
          <p v-else>There are no more recipes</p>
        </div>
      </md-app-content>
    </md-app>
  </div>
</template>

<style>
.md-app {
   max-height: 100vh;
}
.md-drawer {
   width: 230px;
   max-width: calc(100vw - 125px);
}
.md-subheader {
  min-height: 24px !important;
}
.horizontal-scroll .md-list-item-content {
  overflow: auto !important;
  overflow-y: hidden !important;
}
.md-card {
  margin-top: 24px;
}
.child-top-margin > div {
  margin-top: 3px;
}
.yellow {
  color: orange !important;
}
.text-align-left {
  text-align: left;
  padding-left: 24px;
}
@media (hover: none) and (pointer: coarse) {
  ::-webkit-scrollbar {
    display: none;
  }
}
</style>

<script>
import recipes from '../data/recipes.json';

export default {
  name: 'Home',

  data: () => ({
    menuVisible: false,
    searchTextHolder: null,
    searchTerm: null,
    loading: false,
    showCount: 12,
    showCountInterval: 12,
    rankings: [],
    tagTypes: {
      ingredients: '食材', meal_types: '用餐时间', course_types: '分类', cooking_styles: '烹饪方式', nutrition_types: '营养成份'
    },
    selectedTags: {
      ingredients: [],
      meal_types: [],
      course_types: [],
      cooking_styles: [],
      nutrition_types: [],
    },
    selectedTagsMerged: [],
    selectedPreparationTimes: [],
    tags: {
      ingredients: [],
      meal_types: [],
      course_types: [],
      cooking_styles: [],
      nutrition_types: [],
    },
    tagsMerged: [],
    preparation_times: ['<10 min', '10-29 min', '30-60 min', '>60 min'],
    recipes,
  }),

  computed: {
    unselectedTags() {
      let result = {
        ingredients: [],
        meal_types: [],
        course_types: [],
        cooking_styles: [],
        nutrition_types: [],
      };

      return result;
    },
    unselectedPreparationTimes() {
      let result = []
      for (let i = 0; i < this.preparation_times.length; i++) {
        let pt = this.preparation_times[i];
        if (this.selectedPreparationTimes.includes(pt)) { continue; }
        this.preparation_times.push(pt);
      }
    },
  },

  methods: {
    initTags() {
      // compute merged tags for each recipe
      for (let i = 0; i < this.recipes.length; i++) {
        let recipe = this.recipes[i];
        const tags = [];
        for (const tagType in this.tagTypes) {
          tags.push(...recipe[tagType]);
        }
        recipe.tags = tags;
      }
      // collect and sort over tags
      let intermediate = {
        ingredients: {},
        meal_types: {},
        course_types: {},
        cooking_styles: {},
        nutrition_types: {},
      };
      for (let i = 0; i < this.recipes.length; i++) {
        let recipe = this.recipes[i];
        for (const tagType in this.tagTypes) {
          let typedTags = recipe[tagType];
          for (let j = 0; j < typedTags.length; j++) {
            let tag = typedTags[j];
            if (!intermediate[tagType][tag]) { intermediate[tagType][tag] = 0; }
            intermediate[tagType][tag] += 1;
          }
        }
      }
      // sort
      for (const tagType in this.tagTypes) {
        let tags = Object.keys(intermediate[tagType]).map((tag) => ({ tag: tag, count: intermediate[tagType][tag] }));
        tags.sort((a, b) => {
          return b.count - a.count;
        });
        this.tags[tagType] = tags.map((tag) => tag.tag);
      }
      // compute overall merged tags
      this.tagsMerged = [];
      for (const tagType in this.tagTypes) {
        this.tagsMerged.push(...this.tags[tagType]);
      }
    },
    deselectTag(tagType, tag) {
      if (tagType === 'preparation_times') {
        let index = this.selectedPreparationTimes.indexOf(tag);
        if (index !== -1) this.selectedPreparationTimes.splice(index, 1);
        this.updateResults();
        return;
      }
      let index = this.selectedTags[tagType].indexOf(tag);
      if (index !== -1) this.selectedTags[tagType].splice(index, 1);
      this.updateResults();
    },
    selectTag(tagType, tag) {
      if (tagType === 'preparation_times') {
        this.selectedPreparationTimes.push(tag);
        this.updateResults();
        return;
      }
      this.selectedTags[tagType].push(tag);
      this.updateResults();
    },
    updateResults() {
      this.loading = true;
      this.selectedTagsMerged = [];
      for (const tagType in this.tagTypes) {
        this.selectedTagsMerged.push(...this.selectedTags[tagType]);
      }
      this.showCount = this.showCountInterval;
      this.rankings = this.recipes.map((recipe, index) => {
        let score = 0;
        if (this.isInSelectedTimeRange(recipe.preparation_time)) {
          score += 11;
        }
        for (const key in this.selectedTags) {
          for (let i=0; i < recipe[key].length; i++) {
            if (this.selectedTags[key].includes(recipe[key][i])) {
              score += 11;
            }
          }
        }
        score += recipe.rating;
        return { index: index, score: score };
      });
      this.rankings.sort((a, b) => {
        return b.score - a.score;
      });
      this.loading = false;
    },
    loadMore() {
      this.showCount += this.showCountInterval;
    },
    isInSelectedTimeRange(time) {
      for (let i=0; i < this.selectedPreparationTimes.length; i++) {
        let selectedPrepTime = this.selectedPreparationTimes[i];
        switch(selectedPrepTime) {
          case '<10 min':
            if (time < 10) { return true; }
            break;
          case '10-29 min':
            if (time >= 10 && time < 30) { return true; }
            break;
          case '30-60 min':
            if (time >= 30 && time < 60) { return true; }
            break;
          case '>60 min':
            if (time > 60) { return true; }
            break;
          default:
            return false;
        }
      }
      return false;
    },
    openInNew(url) {
      window.open(url, '_blank');
    },
    scrollToTop() {
      this.$el.querySelector('main').scrollTo({ top: 0, behavior: 'smooth' });
    },
    searchChanged(searchTerm) {
      this.searchTerm = searchTerm;
    },
  },

  created() {
    this.loading = true;
    this.initTags();
    this.updateResults();
    this.loading = false;
  },
};
</script>
