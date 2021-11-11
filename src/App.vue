<template>
  <router-view />
</template>

<style lang="scss">
* {
  padding: 0;
  margin: 0;
  box-sizing: border-box;
}
:root {
  --colorD1: #181a1b;
  --colorD2: #202e36;
  --colorD2O5: #202e3683;
  --colorD3: #3e5a69;
  --colorD3O5: #3e5a6983;
  --colorL2: #8a969c;
  --colorL1: #a1aeb5;
  color: var(--colorL1);
  background-color: var(--colorD1);
}
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
}
.main-app-cont .nav-bar {
  height: 5vh;
}
.main-app-cont .router-view {
  height: 95vh;
}
.button1 {
  display: inline-block;
  padding: 5px;
  border-radius: 5px;
  width: fit-content;
  font-weight: bolder;
  color: white;
  background-color: var(--colorD3);
}
.button1:hover {
  opacity: 0.8;
  cursor: pointer;
}
/*Scroll bar*/
::-webkit-scrollbar {
  width: 10px;
  height: 10px;
}
::-webkit-scrollbar-track {
  background: var(--colorD3);
}
::-webkit-scrollbar-thumb {
  background-color: var(--colorL2);
}
::-webkit-scrollbar-thumb:hover {
  background-color: var(--colorL1);
}
</style>
