<script>

</script>

<template>

  <div class="menu">
    <ul>
      <li>
        <div class="menu-icon">

        </div>
      </li>
      <li>
        <div class="menu-icon">
          
        </div>
      </li>
      <li>
        <div class="menu-icon">
          
        </div>
      </li>
    </ul>
  </div>
  
  <div class="toolbar">
    <ul>
        <li><div class="toolbar-icon"></div></li>
        <li><div class="toolbar-icon"></div></li>
        <li><div class="toolbar-icon"></div></li>
    </ul>
  </div>


  <div class="container">
    <div class="first-image-container">
      <h1>Исходное изображение</h1>
      <div class="image">
        <img src="" alt="">
        <div class="statement"></div>
      </div>
    </div>
    <div class="second-image-container">
      <h1>Итоговое изображение</h1>
      <div class="image">
        <img src="" alt="">
        <div class="statement"></div>
      </div>
    </div>
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

  align-items: center;

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

.toolbar ul {
  margin-top: 10px;

  margin-left: 10px;
  display: flex;
  flex-direction: column;

  gap: 10px;
}

.menu-icon {
  width: 50px;
  height: 50px;

  background-color: #000;
}

.toolbar-icon {
  width: 50px;
  height: 50px;

  background-color: #000;
}

.container {
  width: 90vw;
  height: 90vh;

  padding: 50px;

  display: flex;
  flex-direction: row;
  
  border: 1px solid red;

  justify-content: space-between;
}

.first-image-container {

  width: 680px;
  height: 700px;

  display: flex;
  flex-direction: column;

  border: 1px solid green;

  align-items: center;

}

.second-image-container {

  width: 680px;
  height: 700px;

  display: flex;
  flex-direction: column;

  border: 1px solid green;

  align-items: center;
  
}

.image {

  width: 100%;
  height: 100%;

  border: 1px solid red;

  display: flex;
  flex-direction: column;

  position: relative;
}

.image .statement {

  position: absolute;

  top: 85%;
  width: 100%;
  height: 15%;

  border: 1px solid yellow;
}

.container h1 {
  margin-top: 10px;
  margin-bottom: 20px;

  color: #fff;
}

</style>
