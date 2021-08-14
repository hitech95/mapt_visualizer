<template>
  <div class="HelloWorld">
    <img alt="Vue logo" src="../assets/logo.png" />
    <div>{{ msg }}</div>
  </div>
</template>

<script lang="ts">
import { Component, Prop, Vue } from "vue-property-decorator";

@Component
export default class HelloWorld extends Vue {
  @Prop() private msg!: string;
}
</script>
