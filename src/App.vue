<script>
export default {
  name: "App",
  metaInfo: {
    // if no subcomponents specify a metaInfo.title, this title will be used
    title: "Default Title",
    // all titles will be injected into this template
    titleTemplate: "%s | Sky MAP-T Visualizer",
  },
};
</script>

<template>
  <div id="app">
    <nav class="container-fluid">
      <ul>
        <li>
          <router-link to="/">
            <img src="./assets/logo.png"/>
          </router-link>
        </li>
        <li><strong>Sky MAP-T Visualizer</strong></li>
      </ul>
      <ul>
        <li>
          <router-link to="/">Home</router-link>
        </li>
        <li>
          <router-link to="/about">About</router-link>
        </li>
      </ul>
    </nav>

    <div id="content" class="container">
      <router-view/>
      <footer>
        <hr/>
        <p>
          <small>
            <router-link to="/about">Sky MAP-T Visualizer</router-link>
            | Copyright 2021
            <a href="https://getbootstrap.com/" class="secondary">hitech95</a
            ><br/>
            Openly inspired by
            <a
                href="https://forum.fibra.click/d/19642-sky-wifi-passera-a-map-t-entro-fine-anno/287"
                class="secondary"
            >edofullo</a
            >, Licensed under the MIT License
          </small>
        </p>
      </footer>
    </div>
  </div>
</template>

<style lang="scss">
@import "~@picocss/pico/scss/pico";
@import "scss/pico.sky";

#content {
  padding-top: calc(var(--block-spacing-vertical) + #{$navHeight});
}


</style>
