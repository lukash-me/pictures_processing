<script>

</script>

<template>

  <div class="menu">
    <ul>
      <li>
        <div class="icon">

        </div>
      </li>
      <li>
        <div class="icon">
          
        </div>
      </li>
      <li>
        <div class="icon">
          
        </div>
      </li>
    </ul>
  </div>
  
  <div class="toolbar">
    <ul>
        <li>выделить</li>
        <li>скопировать</li>
        <li>вставить</li>
    </ul>
  </div>

</template>

<style scoped>

@import url('https://fonts.googleapis.com/css2?family=Comfortaa:wght@300..700&display=swap');

* {
    margin: 0;
    padding: 0;
    box-sizing: border-box;

    font-family: "Comfortaa", sans-serif;
    font-size: 18px;

    text-decoration: none;
    color: inherit;
    list-style: none;
}

.menu {

  width: 100vw;
  height: 70px;
  background-color: blue;

  top: 0;
  left: 0;
  position: fixed;

  display: flex;

  align-items: center;

  background: #fff;

  z-index: 10;
}

.menu ul {
  
  margin-left: 10px;
  display: flex;
  flex-direction: row;

  gap: 10px;
}

.toolbar {

  display: flex;
  flex-direction: column;

  top: 70px;
  left: 0;

  height: 100vh;
  width: 150px;
  background-color: #000;

  position: fixed;

  color: #fff;
  background: #5c5c5c;

  z-index: 5;
}

.icon {
  width: 50px;
  height: 50px;

  background-color: #000;
}

</style>
