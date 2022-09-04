<template>
  <v-app app>
    <v-app-bar app color="blue" :clipped-left="this.clippedLeft">
      <v-app-bar-nav-icon @click="switchBar" class="white--text"></v-app-bar-nav-icon>
      <v-toolbar-title>{{ AppTitle }}</v-toolbar-title>
    </v-app-bar>
    <v-navigation-drawer app v-model="drawer" :clipped="this.clipped">
      <v-card class="mx-auto my-10" width="200px" height="200px"
        style="border-radius: 50%; overflow: hidden; z-index: 999999999999" @click="handleImgClick" flat>
        <v-img src="./assets/logo.jpg"></v-img>
      </v-card>
      <v-list-item-group mandatory>
        <v-list shaped>
          <v-list-item v-for="item in items" :key="item.title" @click="handleNavListItemClick(item.link)">
            <v-list-item-icon>
              <v-icon>{{ item.icon }}</v-icon>
            </v-list-item-icon>

            <v-list-item-content>
              <v-list-item-title class="ml-6">{{
                  item.title
              }}</v-list-item-title>
            </v-list-item-content>
          </v-list-item>
        </v-list>
      </v-list-item-group>
    </v-navigation-drawer>
    <router-view></router-view>
  </v-app>
</template>

<script>

export default {
  name: 'App',

  data: () => {
    return {
      AppTitle: "Serial",
      drawer: null,
      clippedLeft: true,
      clipped: true,
      items: [
        {
          title: "串口",
          icon: "mdi-serial-port",
          link: "/",
        },
        {
          title: "关于",
          icon: "mdi-information",
          link: "/about",
        },
      ],
    }
  },
  methods: {
    switchBar: function () {
      console.log(`drawer:${this.drawer}`)
      this.drawer = !this.drawer;
    },
    handleImgClick: function () { },
    handleNavListItemClick: function (link) {
      this.$router.replace(link);
    },
  },
};
</script>
