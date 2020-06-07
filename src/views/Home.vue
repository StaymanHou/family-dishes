<template>
  <div class="page-container">
    <md-app md-mode="reveal">
      <md-app-toolbar class="md-primary">
        <md-button class="md-icon-button" @click="menuVisible = !menuVisible">
          <md-icon>menu</md-icon>
        </md-button>
        <span class="md-title">我家的家常菜</span>
      </md-app-toolbar>

      <md-app-drawer :md-active.sync="menuVisible">
        <md-toolbar class="md-transparent" md-elevation="0">Navigation</md-toolbar>

        <md-list>
          <md-list-item>
            <md-icon>move_to_inbox</md-icon>
            <span class="md-list-item-text">Inbox</span>
          </md-list-item>

          <md-list-item>
            <md-icon>send</md-icon>
            <span class="md-list-item-text">Sent Mail</span>
          </md-list-item>

          <md-list-item>
            <md-icon>delete</md-icon>
            <span class="md-list-item-text">Trash</span>
          </md-list-item>

          <md-list-item>
            <md-icon>error</md-icon>
            <span class="md-list-item-text">Spam</span>
          </md-list-item>
        </md-list>
      </md-app-drawer>

      <md-app-content>
        <md-card>
          <md-card-content>
            <md-autocomplete
              class="search"
              v-model="searchTextHolder"
              :md-options="tags.ingredients"
              md-layout="box">
              <label>Search...</label>
            </md-autocomplete>
            <md-divider></md-divider>
            <md-list v-for="(value, key) in tagTypes" class="md-line" :key="key">
              <md-subheader>{{ value }}</md-subheader>
              <md-list-item class="horizontal-scroll">
                <div>
                  <md-chip v-for="tag in tags[key]" :key="tag">{{ tag }}</md-chip>
                </div>
              </md-list-item>
            </md-list>
            <md-list class="md-line">
              <md-subheader>耗时</md-subheader>
              <md-list-item class="horizontal-scroll">
                <div>
                  <md-chip v-for="preparation_time in preparation_times" :key="preparation_time">{{ preparation_time }}</md-chip>
                  <md-chip>Static</md-chip>
                  <md-chip class="md-primary" md-deletable>Deletable</md-chip>
                  <md-chip class="md-accent" md-clickable>Clickable</md-chip>
                  <md-chip md-disabled>Disabled</md-chip>
                </div>
              </md-list-item>
            </md-list>
          </md-card-content>
        </md-card>

        <div style="height: 12px;"></div>

        <div class="md-layout md-alignment-top-space-around">
          <md-card v-for="recipe in recipes" :key="recipe.title" class="md-layout-item md-xlarge-size-20 md-large-size-25 md-medium-size-40 md-small-size-80 md-xsmall-size-90">
            <md-card-header>
              <div class="md-title text-align-left">{{ recipe.title }}</div>
            </md-card-header>

            <md-card-content>
              <div class="text-align-left">
                <md-icon class="yellow" v-for="n in parseInt(recipe.rating/2)">star</md-icon>
                <md-icon class="yellow" v-for="n in recipe.rating%2">star_half</md-icon>
                <md-icon v-for="n in parseInt((10-recipe.rating)/2)">star_border</md-icon>
              </div>
              <div class="child-top-margin">
                <md-chip class="md-primary">{{ recipe.preparation_time }} min</md-chip>
                <md-chip v-for="tag in getTags(recipe)" :key="tag">{{ tag }}</md-chip>
              </div>
            </md-card-content>

            <md-card-actions>
              <md-button class="md-primary" @click="openInNew(recipe.url)">
                <md-icon>open_in_new</md-icon>
              </md-button>
            </md-card-actions>
          </md-card>
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
    tagTypes: {
      ingredients: '食材', meal_types: '用餐时间', course_types: '分类', cooking_styles: '烹饪方式', nutrition_types: '营养成份',
    },
    preparation_times: ['<10 min', '10-30 min', '30-60 min', '>60 min'],
    tags: {
      ingredients: ['土豆', '番茄'],
      meal_types: ['早餐', '午餐', '晚餐', '小吃'],
      course_types: ['主食', '素菜', '小荤', '大荤', '汤菜', '前菜', '点心'],
      cooking_styles: ['炒', '煎', '炖煮', '蒸', '烤', '凉拌', '卤', '炸'],
      nutrition_types: ['低脂', '低糖', '高脂', '高糖'],
    },
    recipes,
  }),

  methods: {
    getTags(recipe) {
      const tags = [];
      for (const tagType in this.tagTypes) {
        tags.push(...recipe[tagType]);
      }
      return tags;
    },
    openInNew(url) {
      window.open(url, '_blank');
    },
  },
};
</script>
