<script>

  export default{

    data: () => ({  
      fileBlob: null,
      fileName: null,
      fileType: null,
      imageSrc: null
    }),

    methods: {

      onFileChange(event) {
        const file = event.target.files[0]; // Получаем файл
        if (!file) return;

        this.fileBlob = file; // Сохраняем файл в переменную
        this.fileName = file.name; // Запоминаем имя файла
        this.fileType = file.type; // Запоминаем формат (MIME type)
        this.imageSrc = URL.createObjectURL(file);
    },

      triggerImgLoad() {
        this.$refs.fileInput.click();
      },

    }
  };

</script>

<template>

  <div class="menu">
    <ul>
      <li>
        <div class="menu-icon">
          <img src="/icon-step-back.png" alt="" title="Шаг назад">
        </div>
      </li>
      <li>
        <div class="menu-icon">
          <img src="/icon-save-img.png" alt="" title="Сохранить изображение">
        </div>
      </li>
      <li>
        <div class="menu-icon" @click="triggerImgLoad">
          <img src="/icon-load-img.png" alt="" title="Загрузить изображение">
          <input type="file" @change="onFileChange" ref="fileInput" accept="*" style="display: none;">
        </div>
      </li>
    </ul>
  </div>
  
  <div class="toolbar">
    <ul>
        <li><div class="toolbar-icon">
          <img src="/icon-select-area.png" alt="" title="Выделить область">
        </div></li>
        <li><div class="toolbar-icon">
          <img src="/icon-copy.png" alt="" title="Скопировать">
        </div></li>
        <li><div class="toolbar-icon">
          <img src="/icon-paste.png" alt="" title="Вставить">
        </div></li>
    </ul>
  </div>


  <div class="container">
    <div class="first-image-container">
      <h1>Исходное изображение</h1>
      <div class="image">
        <img v-if="imageSrc" :src="imageSrc" alt="">
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

  box-shadow: 0px 4px 8px black;
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
  padding-left: 10px;

  top: 70px;
  left: 0;

  height: 100vh;
  width: 150px;
  background-color: #000;

  position: fixed;

  color: #fff;
  background: #5c5c5c;

  box-shadow: 4px 0px 12px rgba(0, 0, 0, 0.47);

  z-index: 5;
}

.toolbar ul {
  margin-top: 20px;

  margin-left: 10px;
  display: flex;
  flex-direction: column;

  gap: 10px;
}

.menu-icon {
  width: 50px;
  height: 50px;

  border: 1px solid #000000;
  border-radius: 4px;

  display: flex;
  justify-content: center;
  align-items: center;
}

.menu-icon img, .toolbar-icon img{
  width: 96%;
  height: 96%;

  overflow: hidden;
}

.toolbar-icon {
  width: 50px;
  height: 50px;

  background-color: #fff;
  border: 1px solid #000000;
  border-radius: 4px;

  display: flex;
  justify-content: center;
  align-items: center;
}

.container {
  width: 90vw;
  height: 90vh;

  padding: 50px;

  display: flex;
  flex-direction: row;

  justify-content: space-between;
}

.first-image-container {

  width: 680px;
  height: 700px;

  display: flex;
  flex-direction: column;

  align-items: center;

}

.second-image-container {

  width: 680px;
  height: 700px;

  display: flex;
  flex-direction: column;

  align-items: center; 
}

.image {

  width: 100%;
  height: 100%;
  
  border: 4px solid rgb(0, 0, 0);
  border-radius: 12px;

  background-color: #b1b1b1;

  overflow: hidden;

  display: flex;
  flex-direction: column;

  position: relative;
}

.image img {
  max-width: 100%;
  max-height: 100%;
  object-fit: contain;
  
}

.image .statement {

  position: absolute;

  top: 85%;
  width: 100%;
  height: 15%;

  border: 2px solid rgba(0, 0, 0, 0.505);
  border-radius: 8px;

  background-color: rgba(150, 150, 150, 0.514);
}

.container h1 {
  margin-top: 10px;
  margin-bottom: 20px;

  color: #fff;
}

</style>
