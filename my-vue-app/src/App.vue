<script>
  import { getTransitionRawChildren, ref } from 'vue'

  export default{  

    data: () => ({  
      chartRef: ref(null),
      chartInstance: null,
      fileBlob: null,
      fileName: null,
      fileType: null,
      imageSrc: null,
      imageResultSrc: null,
      cropCoords: null,
      cropRect: null,
      isCropping: ref(false),
      isStatementSource: ref(false),
      isStatementResult: ref(false),

      sourceHistogramsOpen: ref(false),
      resultHistogramsOpen: ref(false),

      //Для гистограмм
      redSourceCanvas: ref(null),
      greenSourceCanvas: ref(null),
      blueSourceCanvas: ref(null),
      intSourceCanvas: ref(null),

      //Для бинаризации

      isBinary: ref(false),
      firstColor: ref('#000000'),
      secondColor: ref('#ffffff'),
      thresholdFirst: ref(50),

      // Для изменения яркости
      
      isChangeBrightness: ref(false),
      brightness: ref(50),
      contrast: ref(1),

      //Само изображение для обработки
      resultImagesStack: [],
      resultImage: null,
      sourceImage: null,

      //Для профиля

      lineCoords: [null, null, null, null],
      dxdy: [null, null],

      sourceProfileLineOn: ref(false),
      sourceProfileOpen: ref(false),
      sourceDrawProfileLineOpen: ref(false),

      resultProfileLineOn: ref(false),
      resultProfileOpen: ref(false),
      resultDrawProfileLineOpen: ref(false),

      // Шумы

      isNoiseOpen: ref(false),
      noiseProbability: ref(0.1),

      //

      RGB: [0,0,0],
      cursorCoords: [0,0],
      imgSize: [1,1],
    }),

    methods: {

      openDrawProfileLine(imageStage) {

        if (imageStage=='source'){

          if (this.sourceProfileOpen) {
            this.sourceProfileOpen = false;
          }

          this.sourceHistogramsOpen = false;
          this.sourceDrawProfileLineOpen = true;
          this.sourceProfileLineOn = true;
        }

        else if(imageStage=='result'){

          if (this.resultProfileOpen) {
            this.resultProfileOpen = false;
          }

          this.resultHistogramsOpen = false;
          this.resultDrawProfileLineOpen = true;
          this.resultProfileLineOn = true;
        }
      },

      profileProcess(imageStage){

        let canvas, ctx;

        if (imageStage == "result"){

          this.resultDrawProfileLineOpen = false;
          this.resultProfileOpen = true;

          ({ canvas, ctx } = this.makeCanvas('result'));
          ctx.drawImage(this.resultImage, 0, 0);

          const pixels = this.getPixelsOnLine(ctx);

          this.$nextTick(() => {
          const canvas = this.$refs.resultProfileCanvas;
          this.drawProfile(canvas, this.$refs.resultProfileCanvas.getContext('2d'), pixels);
        });
        }

        else if (imageStage == "source"){

          this.sourceDrawProfileLineOpen = false;
          this.sourceProfileOpen = true;

          ({ canvas, ctx } = this.makeCanvas('source'));
          ctx.drawImage(this.sourceImage, 0, 0);

          const pixels = this.getPixelsOnLine(ctx);

          this.$nextTick(() => {
            const canvas = this.$refs.sourceProfileCanvas;
            this.drawProfile(canvas, this.$refs.sourceProfileCanvas.getContext('2d'), pixels);
          });
        }
      },

      drawProfile(canvas, ctx, data){

        const r = data.map(d => d.r);
        const g = data.map(d => d.g);
        const b = data.map(d => d.b);

        ctx.clearRect(0, 0, canvas.width, canvas.height);

        const padding = 40;
        const height = 255;
        const width = 700;
        const steps = 5;
        const maxValue = 255;

        ctx.font = '12px Arial';
        ctx.fillStyle = '#000';
        ctx.textAlign = 'right';
        ctx.textBaseline = 'middle';

        //Y
        ctx.beginPath();
        ctx.moveTo(padding, 0);
        ctx.lineTo(padding, height);
        ctx.strokeStyle = '#000';
        ctx.stroke();

        for (let i = 250; i >=0; i-=50) {
          ctx.fillText(i.toString(), padding - 5, 250-i);
        }

        //X
        ctx.beginPath();
        ctx.moveTo(padding, height);
        ctx.lineTo(width, height);
        ctx.strokeStyle = '#000';
        ctx.stroke();

        ctx.textBaseline = 'top';

        for (let i = 0; i<=4; i++) {
          ctx.fillText(i * 25, width/4*i, 255);
        }

        this.drawChannel(ctx, r, 'red', padding);
        this.drawChannel(ctx, g, 'green', padding);
        this.drawChannel(ctx, b, 'blue', padding);
      },

      drawChannel(ctx, data, color, padding){
        ctx.beginPath();
        ctx.strokeStyle=color;

        ctx.moveTo(padding, 255-data[0]);

        for (let i=1; i<data.length; i++){
          const x = (i / (data.length - 1)) * 700;
          ctx.lineTo(x+padding, 255-data[i]);
        }

        ctx.stroke();
      },

      getPixelsOnLine(ctx){

        const pixels = [];

        const dx = Math.abs(this.lineCoords[2]-this.lineCoords[0]);
        const dy = -Math.abs(this.lineCoords[3]-this.lineCoords[1]);
        const sx = this.lineCoords[0] < this.lineCoords[2] ? 1 : -1;
        const sy = this.lineCoords[1] < this.lineCoords[3] ? 1 : -1;
        let err = dx + dy;

        let x = this.lineCoords[0];
        let y = this.lineCoords[1];

        while (true) {
          const pixel = ctx.getImageData(x, y, 1, 1).data;
          pixels.push({ x, y, r: pixel[0], g: pixel[1], b: pixel[2] });

          if (x === this.lineCoords[2] && y === this.lineCoords[3]) break;
          const e2 = 2 * err;
          if (e2 >= dy) { err += dy; x += sx; }
          if (e2 <= dx) { err += dx; y += sy; }
        }

        return pixels;
      },

      openHistograms(imageStage){

        if (imageStage=='source'){

          // if (this.sourceHistogramsOpen==true){
          //   this.sourceHistogramsOpen=false;
          //   return;
          // }
          if (this.sourceDrawProfileLineOpen){
            this.sourceProfileLineOn = false;
            this.sourceDrawProfileLineOpen = false;
            
          }

          if (this.sourceProfileOpen) {
            this.sourceProfileLineOn = false;
            this.sourceProfileOpen = false
          }

          this.sourceDrawProfileLineOpen=false;
          this.sourceHistogramsOpen=true;
        }
        else if(imageStage=='result'){

          // if (this.resultHistogramsOpen==true){
          //   this.resultHistogramsOpen=false;
          //   return;
          // }

          if (this.resultDrawProfileLineOpen){
            this.resultProfileLineOn = false;
            this.resultDrawProfileLineOpen = false;
          }

          if (this.resultProfileOpen) {
            this.resultProfileLineOn = false;
            this.resultProfileOpen = false
          }

          this.resultDrawProfileLineOpen=false;
          this.resultHistogramsOpen=true;
        }
        
        this.$nextTick(() => {
          this.histogramsProcess(imageStage);
        });

      },

      histogramsProcess(imageStage) {

        let canvas, ctx;

        if(imageStage == "result"){
          ({ canvas, ctx } = this.makeCanvas('result'));
          ctx.drawImage(this.resultImage, 0, 0);
        }
        else if (imageStage == "source"){
          ({ canvas, ctx } = this.makeCanvas('source'));
          ctx.drawImage(this.sourceImage, 0, 0);
        }

        const imageData = ctx.getImageData(0, 0, canvas.width, canvas.height);
        const pixels = imageData.data;

        const { rHist, gHist, bHist } = this.calculateRDBHistograms(pixels);
        const iHist = this.calculateIntensityHistograms(pixels);

        if (imageStage == "result"){
          this.drawHistogram(this.$refs.redResultCanvas.getContext('2d'), rHist, 'red');
          this.drawHistogram(this.$refs.greenResultCanvas.getContext('2d'), gHist, 'green');
          this.drawHistogram(this.$refs.blueResultCanvas.getContext('2d'), bHist, 'blue');
          this.drawHistogram(this.$refs.intResultCanvas.getContext('2d'), iHist, 'black');
        }
        else if (imageStage == "source"){
          this.drawHistogram(this.$refs.redSourceCanvas.getContext('2d'), rHist, 'red');
          this.drawHistogram(this.$refs.greenSourceCanvas.getContext('2d'), gHist, 'green');
          this.drawHistogram(this.$refs.blueSourceCanvas.getContext('2d'), bHist, 'blue');
          this.drawHistogram(this.$refs.intSourceCanvas.getContext('2d'), iHist, 'black');
        }
      },

      calculateRDBHistograms(pixels) {
        const rHist = new Array(256).fill(0);
        const gHist = new Array(256).fill(0);
        const bHist = new Array(256).fill(0);

        for (let i = 0; i < pixels.length; i += 4) {
          rHist[pixels[i]]++
          gHist[pixels[i + 1]]++
          bHist[pixels[i + 2]]++
        }

        return { rHist, gHist, bHist }
      },

      calculateIntensityHistograms(pixels) {
        const iHist = new Array(256).fill(0);

        for (let i = 0; i < pixels.length; i += 4) {

          const intensity = Math.round(this.calcIntensity(pixels[i], pixels[i+1], pixels[i+2]));
          iHist[intensity]++;
        }

        return iHist;
      },

      drawHistogram(ctx, data, color) {

        const width = ctx.canvas.width;
        const height = ctx.canvas.height;
        const padding = 40;

        ctx.clearRect(0, 0, width, height);

        const chartWidth = width - padding;
        const chartHeight = height - padding;

        const max = Math.max(...data);
        const barWidth = chartWidth / data.length;

        ctx.clearRect(0, 0, width, height);

        //Y
        ctx.beginPath();
        ctx.moveTo(padding, 0);
        ctx.lineTo(padding, chartHeight);
        ctx.strokeStyle = '#000';
        ctx.stroke();

        //X
        ctx.beginPath();
        ctx.moveTo(padding, chartHeight);
        ctx.lineTo(width, chartHeight);
        ctx.strokeStyle = '#000';
        ctx.stroke();

        ctx.fillStyle = '#000';
        ctx.font = '12px sans-serif';
        ctx.textAlign = 'center';
        ctx.textBaseline = 'top';

        for (let i = 0; i <= 255; i += 50) {
          const x = padding + i * barWidth;
          ctx.fillText(i.toString(), x, chartHeight + 5);
        }

        ctx.textAlign = 'right';
        ctx.textBaseline = 'middle';
        ctx.fillText(max.toString(), padding-5, 10);

        for (let i = 0; i < data.length; i++) {
          const value = data[i];
          const barHeight = (value / max) * chartHeight;

          ctx.fillStyle = color;
          ctx.fillRect(
            padding + i * barWidth, 
            chartHeight - barHeight, 
            barWidth, 
            barHeight);
        }
      },

      toBinaryImage(){
        this.isBinary = true;
      },

      closeBinary() {
        this.isBinary = false;
      },

      toAddNoise() {
        this.isNoiseOpen = true;
      },

      closeAddNoise() {
        this.isNoiseOpen = false;
      },

      addNoiseProcess() {

        const { canvas, ctx } = this.makeCanvas("result");

        ctx.drawImage(this.resultImage, 0, 0);

        const imageData = ctx.getImageData(0, 0, canvas.width, canvas.height);
        const pixels = imageData.data;

        for (let i=0; i<pixels.length; i+=4) {

          const randomInt = Math.round(Math.random() * 10);

          if (randomInt/10 < this.noiseProbability) {

            if (Math.round(Math.random()) == 0) {
              
              pixels[i] = 0;
              pixels[i+1] = 0;
              pixels[i+2] = 0;
            }
            else {

              pixels[i] = 255;
              pixels[i+1] = 255;
              pixels[i+2] = 255;
            }
          }
        }

        ctx.putImageData(imageData, 0, 0);
        this.putImageToStack(imageData);
        this.makeResultImage(canvas);
      },

      linearFilterProcess() {
        const { canvas, ctx } = this.makeCanvas("result");

        ctx.drawImage(this.resultImage, 0, 0);

        const imageData = ctx.getImageData(0, 0, canvas.width, canvas.height);
        const pixels = imageData.data;

        let pixelsMatrix = this.make2DPixelsArray(pixels, canvas.width, canvas.height);

        let resultPixelsMatrix = this.makeEmpty2DPixelsArray(canvas.width, canvas.height);

        // Проход 3x3. avg в центр

        for (let m=0; m<canvas.height-2; m++){
          for (let i=0; i<canvas.width-2; i++){
            let redFilterValue = 0;
            let greenFilterValue = 0;
            let blueFilterValue = 0;

            for (let j=0; j<3; j++){
              redFilterValue += pixelsMatrix[m+j][i][0] + pixelsMatrix[m+j][i+1][0] + pixelsMatrix[m+j][i+2][0];
              greenFilterValue += pixelsMatrix[m+j][i][1] + pixelsMatrix[m+j][i+1][1] + pixelsMatrix[m+j][i+2][1];
              blueFilterValue += pixelsMatrix[m+j][i][2] + pixelsMatrix[m+j][i+1][2] + pixelsMatrix[m+j][i+2][2];
            }

            resultPixelsMatrix[m+1][i+1][0] = redFilterValue / 9;
            resultPixelsMatrix[m+1][i+1][1] = greenFilterValue / 9;
            resultPixelsMatrix[m+1][i+1][2] = blueFilterValue / 9;
          }
        }

        // Проход 2x2. avg в угол.

        for (let i=0; i<canvas.width-1; i++){

          //[up, down, left, right]

          let filterValue = {
            0: [0, 0, 0, 0],
            1: [0, 0, 0, 0],
            2: [0, 0, 0, 0]
          }

          for (let j=0; j<2; j++){
            for (let k=0; k<3; k++){
              filterValue[k][0] += pixelsMatrix[j][i][k] + pixelsMatrix[j][i+1][k];
              filterValue[k][1] += pixelsMatrix[canvas.height-2+j][i][k] + pixelsMatrix[canvas.height-2+j][i+1][k] ;
              filterValue[k][2] += pixelsMatrix[i][j][k] + pixelsMatrix[i+1][j][k] ;
              filterValue[k][3] += pixelsMatrix[i][canvas.width-2+j][k] + pixelsMatrix[i+1][canvas.width-2+j][k] ;
            }
          }
          
          for (let k=0; k<3; k++) {
            resultPixelsMatrix[0][i][k]               = filterValue[k][0] / 4;
            resultPixelsMatrix[canvas.height-1][i][k] = filterValue[k][1] / 4;
            resultPixelsMatrix[i][0][k]               = filterValue[k][2] / 4;
            resultPixelsMatrix[i][canvas.width-1][k]  = filterValue[k][3] / 4;
          }
        }

        this.make1DPixelsArray(pixels, resultPixelsMatrix, canvas.width, canvas.height);

        ctx.putImageData(imageData, 0, 0);
        this.putImageToStack(imageData);
        this.makeResultImage(canvas);
      },

      medianFilterProcess() {
        const { canvas, ctx } = this.makeCanvas("result");

        ctx.drawImage(this.resultImage, 0, 0);

        const imageData = ctx.getImageData(0, 0, canvas.width, canvas.height);
        const pixels = imageData.data;

        let pixelsMatrix = this.make2DPixelsArray(pixels, canvas.width, canvas.height);

        let resultPixelsMatrix = this.makeEmpty2DPixelsArray(canvas.width, canvas.height);

        // Проход 3x3. median в центр

        for (let m=0; m<canvas.height-2; m++){
          for (let i=0; i<canvas.width-2; i++){
            let redFilterValues = [];
            let greenFilterValues = [];
            let blueFilterValues = [];

            for (let j=0; j<3; j++){
              for (let k=0; k<3; k++){
                redFilterValues.push(pixelsMatrix[m+j][i+k][0]);
                greenFilterValues.push(pixelsMatrix[m+j][i+k][1]);
                blueFilterValues.push(pixelsMatrix[m+j][i+k][2])
              }
            }

            const medians = [this.calculateMedian(redFilterValues), 
                            this.calculateMedian(greenFilterValues), 
                            this.calculateMedian(blueFilterValues)
                          ];

            resultPixelsMatrix[m+1][i+1][0] = medians[0];
            resultPixelsMatrix[m+1][i+1][1] = medians[1];
            resultPixelsMatrix[m+1][i+1][2] = medians[2];

          }
        }

        // Проход 3x3. median в угол.

        for (let i=0; i<canvas.width-2; i++){

          //[up, down, left, right]

          let filterValue = {
            0: [[], [], [], []],
            1: [[], [], [], []],
            2: [[], [], [], []]
          }

          for (let j=0; j<3; j++){
            for (let k=0; k<3; k++){
              for(let m=0; m<3; m++) {
                filterValue[k][0].push(pixelsMatrix[j][i+m][k]);
                filterValue[k][1].push(pixelsMatrix[canvas.height-3+j][i+m][k]);
                filterValue[k][2].push(pixelsMatrix[i+m][j][k]);
                filterValue[k][3].push(pixelsMatrix[i+m][canvas.width-3+j][k]);
              }
            }
          }

          for (let k=0; k<3; k++) {
            resultPixelsMatrix[0][i][k]               = this.calculateMedian(filterValue[k][0]);
            resultPixelsMatrix[canvas.height-1][i][k] = this.calculateMedian(filterValue[k][1]);
            resultPixelsMatrix[i][0][k]               = this.calculateMedian(filterValue[k][2]);
            resultPixelsMatrix[i][canvas.width-1][k]  = this.calculateMedian(filterValue[k][3]);
          }
        }

        this.make1DPixelsArray(pixels, resultPixelsMatrix, canvas.width, canvas.height);

        ctx.putImageData(imageData, 0, 0);
        this.putImageToStack(imageData);
        this.makeResultImage(canvas);
      },

      calculateMedian(array) {
        array.sort((a, b) => a - b);
        return array[4];
      },

      statisticsMethodProcess() {
        const { canvas, ctx } = this.makeCanvas("result");

        ctx.drawImage(this.resultImage, 0, 0);

        const imageData = ctx.getImageData(0, 0, canvas.width, canvas.height);
        const pixels = imageData.data;

        const intensityArray = this.makeIntensityArray(pixels);
        let intensityMatrix = this.make2DIntensityArray(intensityArray, canvas.width, canvas.height);

        let resultIntensityMatrix = Array.from({ length: canvas.height }, () => Array(canvas.width).fill(0));


        for (let i=0; i<canvas.height-2; i++) {
          for (let j=0; j<canvas.width-2; j++) {

            let sum = 0;
            let A = [];

            for (let m=0; m<2; m++) {
              for (let k=0; k<2; k++) {
                A.push(intensityMatrix[i+m][j+k]);
                sum += A[A.length-1]; 
              }
            }

            const mu = sum / 9;

            sum = 0;

            for (let m=0; m<A.length; m++) {
              sum += Math.pow((A[m] - mu), 2) 
            }

            const sigma = Math.abs(sum / 9)

            let F = sigma * intensityMatrix[i+1][j+1]/ 1000;

            if (F > 255) {
              F = 255;
            }

            resultIntensityMatrix[i+1][j+1] = F;
          }
        }


        this.make1DPixelsArrayFromIntensity(pixels, resultIntensityMatrix, canvas.width, canvas.height);

        ctx.putImageData(imageData, 0, 0);
        this.putImageToStack(imageData);
        this.makeResultImage(canvas);
      },

      uallasProcess() {
        const { canvas, ctx } = this.makeCanvas("result");

        ctx.drawImage(this.resultImage, 0, 0);

        const imageData = ctx.getImageData(0, 0, canvas.width, canvas.height);
        const pixels = imageData.data;

        const intensityArray = this.makeIntensityArray(pixels);
        let intensityMatrix = this.make2DIntensityArray(intensityArray, canvas.width, canvas.height);

        let resultIntensityMatrix = Array.from({ length: canvas.height }, () => Array(canvas.width).fill(0));

        for (let i=0; i<canvas.height-2; i++) {
          for (let j=0; j<canvas.width-2; j++) {

            let A = [];

            A.push(intensityMatrix[i][j+1]);
            A.push(intensityMatrix[i+1][j]);
            A.push(intensityMatrix[i+1][j+2]);
            A.push(intensityMatrix[i+2][j+1]);

            let F = intensityMatrix[i+1][j+1];

            let multiply = 1;

            for (let i=0; i<A.length; i++) {
              if (A[i] == 0) {
                A[i] = 1;
              }

              multiply *= F / A[i];
            }

            F = Math.log(multiply) / 4 * 1000 + 50;

            if (F > 255) {
              F = 255;
            }
            else if (F < 0) {
              F = 0;
            }

            resultIntensityMatrix[i+1][j+1] = F;
          }
        }

        this.make1DPixelsArrayFromIntensity(pixels, resultIntensityMatrix, canvas.width, canvas.height);

        ctx.putImageData(imageData, 0, 0);
        this.putImageToStack(imageData);
        this.makeResultImage(canvas);
      },

      sobelProcess() {
        const { canvas, ctx } = this.makeCanvas("result");

        ctx.drawImage(this.resultImage, 0, 0);

        const imageData = ctx.getImageData(0, 0, canvas.width, canvas.height);
        const pixels = imageData.data;

        const intensityArray = this.makeIntensityArray(pixels);
        let intensityMatrix = this.make2DIntensityArray(intensityArray, canvas.width, canvas.height);

        let resultIntensityMatrix = Array.from({ length: canvas.height }, () => Array(canvas.width).fill(0));

        for (let i=0; i<canvas.height-2; i++) {
          for (let j=0; j<canvas.width-2; j++) {

            let A = [];

            for (let k=0; k<3; k++) {
              for (let m=0; m<3; m++) {
                A.push(intensityMatrix[i+k][j+m]);
              }
            }

            let F = 0;

            const X = (A[0] + 2*A[1] + A[2]) - (A[6] + 2*A[7] + A[8]);
            const Y = (A[0] + 2*A[3] + A[6]) - (A[2] + 2*A[5] + A[8]);

            F = Math.sqrt(Math.pow(X, 2) + Math.pow(Y, 2));

            if (F > 255) {
              F = 255;
            }
            else if (F < 0) {
              F = 0;
            }

            resultIntensityMatrix[i+1][j+1] = F;
          }
        }

        this.make1DPixelsArrayFromIntensity(pixels, resultIntensityMatrix, canvas.width, canvas.height);

        ctx.putImageData(imageData, 0, 0);
        this.putImageToStack(imageData);
        this.makeResultImage(canvas);
      },

      robertsProcess() {
        const { canvas, ctx } = this.makeCanvas("result");

        ctx.drawImage(this.resultImage, 0, 0);

        const imageData = ctx.getImageData(0, 0, canvas.width, canvas.height);
        const pixels = imageData.data;

        const intensityArray = this.makeIntensityArray(pixels);
        let intensityMatrix = this.make2DIntensityArray(intensityArray, canvas.width, canvas.height);

        let resultIntensityMatrix = Array.from({ length: canvas.height }, () => Array(canvas.width).fill(0));

        for (let i=0; i<canvas.height-1; i++) {
          for (let j=0; j<canvas.width-1; j++) {

            let A = [];

            for (let k=0; k<2; k++) {
              for (let m=0; m<2; m++) {
                A.push(intensityMatrix[i+k][j+m]);
              }
            }

            let F = Math.sqrt(Math.pow((A[0] - A[3]), 2) + Math.pow((A[1] - A[2]), 2));

            if (F > 255) {
              F = 255;
            }
            else if (F < 0) {
              F = 0;
            }

            resultIntensityMatrix[i][j] = F;
          }
        }

        this.make1DPixelsArrayFromIntensity(pixels, resultIntensityMatrix, canvas.width, canvas.height);

        ctx.putImageData(imageData, 0, 0);
        this.putImageToStack(imageData);
        this.makeResultImage(canvas);
      },

      laplaceProcess() {
        const { canvas, ctx } = this.makeCanvas("result");

        ctx.drawImage(this.resultImage, 0, 0);

        const imageData = ctx.getImageData(0, 0, canvas.width, canvas.height);
        const pixels = imageData.data;

        const intensityArray = this.makeIntensityArray(pixels);
        let intensityMatrix = this.make2DIntensityArray(intensityArray, canvas.width, canvas.height);

        let resultIntensityMatrix = Array.from({ length: canvas.height }, () => Array(canvas.width).fill(0));


        for (let i=0; i<canvas.height-2; i++) {
          for (let j=0; j<canvas.width-2; j++) {

            let A = [];

            for (let k=0; k<3; k++) {
              for (let m=0; m<3; m++) {
                A.push(intensityMatrix[i+k][j+m]);
              }
            }

            const kernel = [-1, -2, -1,
                            -2, 12, -2,
                            -1, -2, -1
            ] 

            let F = 0;

            for (let k=0; k<9; k++) {
              F += A[k] * kernel[k];
            }

            if (F > 255) {
              F = 255;
            }
            else if (F < 0) {
              F = 0;
            }

            resultIntensityMatrix[i+1][j+1] = F;
          }
        }

        this.make1DPixelsArrayFromIntensity(pixels, resultIntensityMatrix, canvas.width, canvas.height);

        ctx.putImageData(imageData, 0, 0);
        this.putImageToStack(imageData);
        this.makeResultImage(canvas);
      },

      kirschProcess() {

        const { canvas, ctx } = this.makeCanvas("result");

        ctx.drawImage(this.resultImage, 0, 0);

        const imageData = ctx.getImageData(0, 0, canvas.width, canvas.height);
        const pixels = imageData.data;

        const intensityArray = this.makeIntensityArray(pixels);
        let intensityMatrix = this.make2DIntensityArray(intensityArray, canvas.width, canvas.height);

        let resultIntensityMatrix = Array.from({ length: canvas.height }, () => Array(canvas.width).fill(0));

        for (let i=0; i<canvas.height-2; i++) {
          for (let j=0; j<canvas.width-2; j++) {
            let A = [
                      intensityMatrix[i][j],
                      intensityMatrix[i][j+1],
                      intensityMatrix[i][j+2],
                      intensityMatrix[i+1][j+2],
                      intensityMatrix[i+2][j+2],
                      intensityMatrix[i+2][j+1],
                      intensityMatrix[i+2][j],
                      intensityMatrix[i+1][j]
                    ];

            let S = [];
            let T = [];

            for (let m=0; m<8; m++) {
              S.push(A[m] + A[this.addMod8(m, 1)] + A[this.addMod8(m, 2)]);

              let sumT = 0;

              for (let p=3; p<8; p++) {
                sumT += A[this.addMod8(m, p)]
              }

              T.push(sumT);
            }

            let ST = [];

            for (let m=0; m<8; m++) {
              ST.push(Math.abs(5*S[m] - 3*T[m]));
            }
            let maxST = Math.max(...ST);

            if (maxST > 255) {
              maxST = 255;
            }

            resultIntensityMatrix[i+1][j+1] = maxST;
          }
        }

        this.make1DPixelsArrayFromIntensity(pixels, resultIntensityMatrix, canvas.width, canvas.height);

        ctx.putImageData(imageData, 0, 0);
        this.putImageToStack(imageData);
        this.makeResultImage(canvas);
      },

      addMod8(a, b) {
        const sum = a + b;
        if (sum > 7) {
          return sum - 8;
        }
        return sum;
      },

      make2DIntensityArray(intensityArray, width, height) {
        let matrix = [];

        for (let y = 0; y < height; y++) {
          const row = [];
          for (let x = 0; x < width; x++) {
            const index = (y * width + x);
            const pixel = intensityArray[index];
            row.push(pixel);
          }
          matrix.push(row);
        }

        return matrix;
      },

      make2DPixelsArray(pixels, width, height) {

        let matrix = [];

        for (let y = 0; y < height; y++) {
          const row = [];
          for (let x = 0; x < width; x++) {
            const index = (y * width + x) * 4;
            const pixel = {
              0: pixels[index],
              1: pixels[index+1],
              2: pixels[index+2],
              3: pixels[index+3],
            };
            row.push(pixel);
          }
          matrix.push(row);
        }

        return matrix;
      },

      makeEmpty2DPixelsArray(width, height) {

        let matrix = [];

        for (let y = 0; y < height; y++) {
          const row = [];
          for (let x = 0; x < width; x++) {
            const index = (y * width + x) * 4;
            const pixel = {
              0: 0,
              1: 0,
              2: 0,
              3: 255,
            };
            row.push(pixel);
          }
          matrix.push(row);
        }

        return matrix;
      },

      make1DPixelsArrayFromIntensity(pixels, matrix, width, height) {
        for (let y = 0; y < height; y++) {
          for (let x = 0; x < width; x++) {
            const index = (y * width + x) * 4;
            const pixel = matrix[y][x];

            pixels[index]     = pixel;
            pixels[index + 1] = pixel;
            pixels[index + 2] = pixel;
            pixels[index + 3] = 255;
          }
        }
      },

      make1DPixelsArray(pixels, matrix, width, height) {

        for (let y = 0; y < height; y++) {
          for (let x = 0; x < width; x++) {
            const index = (y * width + x) * 4;
            const pixel = matrix[y][x];

            pixels[index]     = pixel[0];
            pixels[index + 1] = pixel[1];
            pixels[index + 2] = pixel[2];
            pixels[index + 3] = pixel[3];
          }
        }
      },

      toChangeBrightness() {
        this.isChangeBrightness = true;
      },

      closeChangeBrightness() {
        this.isChangeBrightness = false;
      },

      negativeProcess() {
        const { canvas, ctx } = this.makeCanvas("source");
        
        ctx.drawImage(this.sourceImage, 0, 0);

        const imageData = ctx.getImageData(0, 0, canvas.width, canvas.height);
        const pixels = imageData.data;

        for (let i=0; i<pixels.length; i+=4){
          pixels[i] = 255 - pixels[i];
          pixels[i+1] = 255 - pixels[i+1];
          pixels[i+2] = 255 - pixels[i+2];
        }

        ctx.putImageData(imageData, 0, 0);
        this.putImageToStack(imageData);
        this.makeResultImage(canvas, imageData);
      },

      shadesOfGrayProcess() {

        const { canvas, ctx } = this.makeCanvas("source");
        
        ctx.drawImage(this.sourceImage, 0, 0);

        const imageData = ctx.getImageData(0, 0, canvas.width, canvas.height);
        const pixels = imageData.data;

        const intensityArr = this.makeIntensityArray(pixels);

        for (let i=0, j=0; i<pixels.length; i+=4, j++){
          pixels[i] = intensityArr[j];
          pixels[i+1] = intensityArr[j];
          pixels[i+2] = intensityArr[j];
        }

        ctx.putImageData(imageData, 0, 0);
        this.putImageToStack(imageData);
        this.makeResultImage(canvas, imageData);
      },

      async binaryProcess() {

        const firstColor = this.hexToRgb(this.firstColor);
        const secondColor = this.hexToRgb(this.secondColor);

        const { canvas, ctx } = this.makeCanvas("source");
        
        ctx.drawImage(this.sourceImage, 0, 0);

        const imageData = ctx.getImageData(0, 0, canvas.width, canvas.height);
        const pixels = imageData.data;

        const intensityArr = this.makeIntensityArray(pixels);

        let minI = Infinity;
        let maxI = -Infinity;

        for (let i=0; i<intensityArr.length; i++){
          if (intensityArr[i] < minI) minI = intensityArr[i];
          if (intensityArr[i] > maxI) maxI = intensityArr[i];
        }

        const thresholdI = this.thresholdFirst;
      
        for (let i=0, j=0; i < pixels.length; i+=4, j++) {
          if (intensityArr[j] >= thresholdI) {
            pixels[i] = secondColor[0];
            pixels[i+1] = secondColor[1];
            pixels[i+2] = secondColor[2];
          }

          else if (intensityArr[j] < thresholdI) {
            pixels[i] = firstColor[0];
            pixels[i+1] = firstColor[1];
            pixels[i+2] = firstColor[2];
          }
        }

        ctx.putImageData(imageData, 0, 0);
        this.putImageToStack(imageData);
        this.makeResultImage(canvas, imageData);
      },

      changeBrightnessContrastProcess() {

        const { canvas, ctx } = this.makeCanvas("result");

        ctx.drawImage(this.resultImage, 0, 0);

        const imageData = ctx.getImageData(0, 0, canvas.width, canvas.height);
        const pixels = imageData.data;

        let r_intensity = 0;
        let g_intensity = 0;
        let b_intensity = 0;

        // Яркость
        for (let i=0; i<pixels.length; i+=4) {
          for (let j = 0; j<3; j++){

            let newPixel = pixels[i+j] + parseInt(this.brightness);
            if (newPixel > 255) {
              newPixel = 255;
            }
            else if (newPixel < 0){
              newPixel = 0;
            }
            pixels[i+j] = newPixel;
          }
          r_intensity += pixels[i];
          g_intensity += pixels[i+1];
          b_intensity += pixels[i+2];
        }

        const channelAmount = pixels.length / 4;
        const avgRGB = [r_intensity / channelAmount, g_intensity / channelAmount, b_intensity / channelAmount];

        // Контрастность
        for (let i=0; i<pixels.length; i+=4) {
          for (let j = 0; j<3; j++){

            let newPixel = this.contrast*(pixels[i+j] - avgRGB[j]) + avgRGB[j];
            if (newPixel > 255) {
              newPixel = 255;
            }
            else if (newPixel < 0){
              newPixel = 0;
            }
            pixels[i+j] = newPixel;
          }
        }

        ctx.putImageData(imageData, 0, 0);
        this.putImageToStack(imageData);
        this.makeResultImage(canvas);
      },

      putImageToStack(imageData) {
        this.resultImagesStack.push(imageData);
      },

      makeIntensityArray(pixels) {
        const size = pixels.length / 4;
        const arr = new Array(size);

        for (let i=0, j=0; i < pixels.length; i+=4, j++) {
          arr[j] = this.calcIntensity(pixels[i], pixels[i+1], pixels[i+2]);
        }

        return arr;
      },

      calcIntensity(r, g, b) {
        return 0.3*r + 0.59*g + 0.11*b;
      },

      makeCanvas(imageStage) {

        const canvas = document.createElement('canvas');

        if(imageStage=='source'){
          canvas.width = this.sourceImage.width;
          canvas.height = this.sourceImage.height;
        }

        else if(imageStage=='result') {
          canvas.width = this.resultImage.width;
          canvas.height = this.resultImage.height;
        }

        const ctx = canvas.getContext("2d");

        return { canvas, ctx };
      },

      async makeResultImage(canvas) {
        const blob = await new Promise((resolve) => {
          canvas.toBlob((blob) => resolve(blob), "image/png");
        });

        this.imageResultSrc = URL.createObjectURL(blob);
        this.resultImage = await this.loadImage(this.imageResultSrc);
      },

      async stepBack() {

        const { canvas, ctx } = this.makeCanvas("result");

        if (this.resultImagesStack.length < 2) {
          return;
        }

        this.resultImagesStack.pop();

        const previousImage = this.resultImagesStack[this.resultImagesStack.length-1];

        ctx.putImageData(previousImage, 0, 0);
        this.makeResultImage(canvas);
      },

      async onFileChange(event) {
        const file = event.target.files[0]; // Получить файл
        if (!file) return;

        this.fileBlob = file;
        this.fileName = file.name;
        this.fileType = file.type;

        this.imageSrc = URL.createObjectURL(file);
        const src = this.imageSrc;
        
        this.sourceImage = await this.loadImage(src)
        this.imgSize = [this.sourceImage.width, this.sourceImage.height];
      },

      loadImage(src) {
        return new Promise((resolve) => {
          const img = new Image();
          img.src = src;
          img.onload = () => {
            resolve(img);
          };
        });
      },

      triggerImgLoad() {
        this.$refs.fileInput.click();
      },

      startCrop(event) {

        if (this.sourceDrawProfileLineOpen == true) {
          this.sourceProfileLineOn = true;
          this.drawProfileLine('source', event);
        }

        if (this.resultDrawProfileLineOpen == true) {
          this.resultProfileLineOn = true;
          this.drawProfileLine('result', event);
        }

        if (this.isCropping == false) {
          return;
        }

        event.preventDefault();

        this.cropRect = this.$refs.sourceImage.getBoundingClientRect();
        this.cropCoords = [event.clientX - this.cropRect.left, event.clientY - this.cropRect.top];

        document.addEventListener("mousemove", this.moveCrop);
        document.addEventListener("mouseup", this.stopCrop);
      },

      drawProfileLine(imageStage, event){

        if (imageStage == "source"){

          event.preventDefault();

          this.cropRect = this.$refs.sourceImage.getBoundingClientRect();
          this.lineCoords = [Math.round(event.clientX - this.cropRect.left), Math.round(event.clientY - this.cropRect.top)];

          document.addEventListener("mousemove", this.moveProfileLine);
          document.addEventListener("mouseup", this.stopProfileLine);
        }
        else if (imageStage == "result"){
          event.preventDefault();

          this.cropRect = this.$refs.resultImage.getBoundingClientRect();
          this.lineCoords = [Math.round(event.clientX - this.cropRect.left), Math.round(event.clientY - this.cropRect.top)];

          document.addEventListener("mousemove", this.moveProfileLine);
          document.addEventListener("mouseup", this.stopProfileLine);
        }
      },

      moveProfileLine(){
        this.lineCoords[2] = this.cursorCoords[0];
        this.lineCoords[3] = this.cursorCoords[1];
      },

      stopProfileLine(){
        document.removeEventListener("mousemove", this.moveProfileLine);
        document.removeEventListener("mouseup", this.stopProfileLine);
      },

      moveCrop(event){

        this.cropCoords[2] = event.clientX - this.cropRect.left;
        this.cropCoords[3] = event.clientY - this.cropRect.top;
      },

      stopCrop(){
        document.removeEventListener("mousemove", this.moveCrop);
        document.removeEventListener("mouseup", this.moveCrop);
      },

      copyAll() {

        this.cropRect = this.$refs.sourceImage.getBoundingClientRect();
        this.isCropping = true;
        this.cropCoords = [0, 0, this.imgSize[0], this.imgSize[1]];
      },

      async cropImage() {

        const left = this.cropCoords[0];
        const top = this.cropCoords[1];
        const width = Math.abs(this.cropCoords[2] - this.cropCoords[0]);
        const height = Math.abs(this.cropCoords[3] - this.cropCoords[1]);
        
        const canvas = document.createElement('canvas');

        canvas.width = width;
        canvas.height = height;

        const ctx = canvas.getContext("2d");

        ctx.drawImage(this.sourceImage, left, top, width, height, 0, 0, width, height);

        const blob = await new Promise((resolve) => {
          canvas.toBlob((blob) => resolve(blob), "image/png");
        });

        this.imageResultSrc = URL.createObjectURL(blob);
        this.resultImage = await this.loadImage(this.imageResultSrc);

        const imageData = ctx.getImageData(0, 0, canvas.width, canvas.height);

        this.resultImagesStack.push(imageData);

        this.clearSelection();
      },

      clearSelection(){

        this.cropCoords = null;

        if (this.isCropping == false){
          this.isCropping = true;
        }

        else if (this.isCropping == true) {
          this.isCropping = false;
        }
      },

      displaySourcePixelColor(event){
        const canvas = document.createElement('canvas');
        const ctx = canvas.getContext('2d');

        canvas.width = this.sourceImage.width;
        canvas.height = this.sourceImage.height;

        ctx.drawImage(this.sourceImage, 0, 0)

        const rect = this.$refs.sourceImage.getBoundingClientRect();

        const x = Math.floor(event.clientX - rect.left);
        const y = Math.floor(event.clientY - rect.top);

        this.cursorCoords = [x, y];

        const pixel = ctx.getImageData(x, y, 1, 1).data;
        this.RGB = pixel;
      },

      displayResultPixelColor(event){
        const canvas = document.createElement('canvas');
        const ctx = canvas.getContext('2d');

        canvas.width = this.resultImage.width;
        canvas.height = this.resultImage.height;

        ctx.drawImage(this.resultImage, 0, 0)

        const rect = this.$refs.resultImage.getBoundingClientRect();

        const x = Math.floor(event.clientX - rect.left);
        const y = Math.floor(event.clientY - rect.top);

        this.cursorCoords = [x, y];

        const pixel = ctx.getImageData(x, y, 1, 1).data;
        this.RGB = pixel;
      },

      startDisplaySourcePixelColor() {
        this.isStatementSource = true
      },

      stopDisplaySourcePixelColor(){
        this.isStatementSource = false;
      },

      startDisplayResultPixelColor() {
        this.isStatementResult = true
      },

      stopDisplayResultPixelColor(){
        this.isStatementResult = false;
      },

      downloadImage(){
        if(!this.resultImage){
          return
        }

        const link = document.createElement("a");
        link.href = this.imageResultSrc;
        link.download = "result-image.png";
        link.click();
      },

      hexToRgb(hex) {

      hex = hex.replace(/^#/, '');

      if (hex.length === 3) {
        hex = hex.split('').map(c => c + c).join('');
      }

      const bigint = parseInt(hex, 16);
      const r = (bigint >> 16) & 255;
      const g = (bigint >> 8) & 255;
      const b = bigint & 255;

      return [r, g, b]
      },
    },

    computed: {
      cropBoxStyle() {
        if (this.cropCoords == null){
          return {
            left: `${0}px`,
            top: `${0}px`,
            width: `${0}px`,
            height: `${0}px`,
          };
        }
        else{
          return {
            left: `${this.cropCoords[0]}px`,
            top: `${this.cropCoords[1]}px`,
            width: `${Math.abs(this.cropCoords[2] - this.cropCoords[0])}px`,
            height: `${Math.abs(this.cropCoords[3] - this.cropCoords[1])}px`,
          };
        }
      },

      profileLineStyle() {

        if (this.lineCoords[0] == null || this.lineCoords[1] == null) {
          return{
            display: 'none'
          };
        }

        this.dxdy[0] = this.lineCoords[2] - this.lineCoords[0];
        this.dxdy[1] = this.lineCoords[3] - this.lineCoords[1];
        
        const length = Math.sqrt(this.dxdy[0] * this.dxdy[0] + this.dxdy[1] * this.dxdy[1]);
        const angle = Math.atan2(this.dxdy[1], this.dxdy[0]) * (180 / Math.PI);

        return {
          left: `${this.lineCoords[0]}px`,
          top: `${this.lineCoords[1]}px`,
          width: `${length}px`,
          transform: `rotate(${angle}deg)`,
          transformOrigin: `0 0`
        };
      },

      correctImgContainer(){
        return{
          width: `${this.imgSize[0]*0.9}px`,
          height: `${this.imgSize[1]*0.9}px`
        }
      },

      statementPosition(){
        return {
          left: `${this.cursorCoords[0]+10}px`,
          top: `${this.cursorCoords[1]+10}px`,
        }
      }
    },

    
  };

</script>

<template>

  <div class="menu">
    <ul>
      <li>
        <div @click="stepBack" class="menu-icon">
          <img src="/icon-step-back.png" alt="" title="Шаг назад">
        </div>
      </li>
      <li>
        <div @click="downloadImage" class="menu-icon">
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
        <li @click="clearSelection"><div class="toolbar-icon">
          <img src="/icon-select-area.png" alt="" title="Выделить область">
        </div></li>
        <li @click="copyAll"><div class="toolbar-icon">
          <img src="/icon-select-all.png" alt="" title="Выделить все">
        </div></li>
        <li @click="cropImage"><div class="toolbar-icon">
          <img src="/icon-paste.png" alt="" title="Вставить">
        </div></li>
        <li @click="toBinaryImage"><div class="toolbar-icon">
          <img src="/icon-binary.png" alt="" title="Бинаризация">
        </div></li>
        <li @click="shadesOfGrayProcess"><div class="toolbar-icon">
          <img src="/icon-gray.png" alt="" title="Оттенки серого">
        </div></li>
        <li @click="negativeProcess"><div class="toolbar-icon">
          <img src="/icon-negative.png" alt="" title="Негатив">
        </div></li>
        <li @click="toChangeBrightness"><div class="toolbar-icon">
          <img src="/icon-brightness.png" alt="" title="Яркость/Контрастность">
        </div></li>
        <li @click="toAddNoise"><div class="toolbar-icon">
          <img src="/icon-noise.png" alt="" title="Добавить шум">
        </div></li>
        <li><div class="toolbar-selector">
          <img src="/icon-filter.png" alt="" title="Фильтр">
          <div class="filters">
            <span @click="linearFilterProcess">Линейный сглаживающий</span>
            <span @click="medianFilterProcess">Нелинейный медианный</span>
            <span @click="kirschProcess">Кирша</span>
            <span @click="laplaceProcess">Лапласа</span>
            <span @click="robertsProcess">Робертса</span>
            <span @click="sobelProcess">Собела</span>
            <span @click="uallasProcess">Уоллеса</span>
            <span @click="statisticsMethodProcess">Статистический метод</span>
          </div>
        </div></li>
    </ul>
  </div>


  <div class="container">
    <div class="first-image-container">
      <h1>Исходное изображение</h1>
      <div class="image" :style="correctImgContainer">
        <img v-if="imageSrc" @mousemove="displaySourcePixelColor" @mouseover="startDisplaySourcePixelColor" @mouseleave="stopDisplaySourcePixelColor" @mousedown="startCrop" :src="imageSrc" ref="sourceImage" alt="" draggable="false">
        <div v-if="isCropping == true" class="crop-box" :style="cropBoxStyle"></div>
        <div v-if="sourceProfileLineOn == true" class="profile-line" :style="profileLineStyle"></div>
        <div v-if="isStatementSource" class="statement" :style="statementPosition">
          <table>
            <tbody>
              <tr class="red">
                <td>R:</td>
                <td>{{ RGB[0] }}</td>
              </tr>
              <tr class="green">
                <td>G:</td>
                <td>{{ RGB[1] }}</td>
              </tr>
              <tr class="blue">
                <td>B:</td>
                <td>{{ RGB[2] }}</td>
              </tr>
            </tbody>
          </table>
        </div>
      </div>
      <div class="btns-container">
        <button class="graphs-btn" @click="openHistograms('source')" title="Построить гистограммы"></button>
        <button class="brightness-profile-btn" @click="openDrawProfileLine('source')" title="Построить профиль яркости"></button>
      </div>
      <div v-if="sourceHistogramsOpen" class="histograms">

        <div class="histsRow"> 
          <div class="redHist">
            <canvas ref="redSourceCanvas" width="400" height="200"></canvas>
          </div>
          <div class="greenHist">
            <canvas ref="greenSourceCanvas" width="400" height="200"></canvas>
          </div>
          <div class="blueHist">
            <canvas ref="blueSourceCanvas" width="400" height="200"></canvas>
          </div>
        </div>
        
        <div class="histsRow">
          <div class="intHist">
            <canvas ref="intSourceCanvas" width="400" height="200"></canvas>
          </div>
        </div>

      </div>
      <div v-if="sourceDrawProfileLineOpen" class="drawProfileLine">
        <span>Начертите линию на фото</span>
        <button @click="profileProcess('source')">Готово</button>
      </div>
      <div v-if="sourceProfileOpen" class="profile">
        <canvas ref="sourceProfileCanvas" width="700" height="400"></canvas>
      </div>
    </div>
    <div class="second-image-container">
      <h1>Итоговое изображение</h1>
      <div class="image" :style="correctImgContainer">
        <img v-if="resultImage" @mousemove="displayResultPixelColor" @mouseover="startDisplayResultPixelColor" @mouseleave="stopDisplayResultPixelColor" @mousedown="startCrop" :src="imageResultSrc" ref="resultImage" alt="" draggable="false">
        <div v-if="resultProfileLineOn == true" class="profile-line" :style="profileLineStyle"></div>
        <div v-if="isStatementResult" class="statement" :style="statementPosition">
          <table>
            <tbody>
              <tr class="red">
                <td>R:</td>
                <td>{{ RGB[0] }}</td>
              </tr>
              <tr class="green">
                <td>G:</td>
                <td>{{ RGB[1] }}</td>
              </tr>
              <tr class="blue">
                <td>B:</td>
                <td>{{ RGB[2] }}</td>
              </tr>
            </tbody>
          </table>
        </div>
      </div>
      <div class="btns-container">
        <button class="graphs-btn" @click="openHistograms('result')" title="Построить гистограммы"></button>
        <button class="brightness-profile-btn" @click="openDrawProfileLine('result')" title="Построить профиль яркости"></button>
      </div>
      <div v-if="resultHistogramsOpen" class="histograms">
        <div class="histsRow">
          <div class="redHist">
            <canvas ref="redResultCanvas" width="400" height="200"></canvas>
          </div>
          <div class="greenHist">
            <canvas ref="greenResultCanvas" width="400" height="200"></canvas>
          </div>
          <div class="blueHist">
            <canvas ref="blueResultCanvas" width="400" height="200"></canvas>
          </div>
        </div>

        <div class="histsRow">
          <div class="intHist">
            <canvas ref="intResultCanvas" width="400" height="200"></canvas>
          </div>
        </div>
      </div>
      <div v-if="resultDrawProfileLineOpen" class="drawProfileLine">
        <span>Начертите линию на фото</span>
        <button @click="profileProcess('result')">Готово</button>
      </div>
      <div v-if="resultProfileOpen" class="profile">
        <canvas ref="resultProfileCanvas" width="700" height="400"></canvas>
      </div>
    </div>

    <div v-if="isBinary" class="binary-options">
      <span>Выберите порог бинаризации: {{ thresholdFirst }}</span>
      <input
        type="range"
        id="threshold"
        min="0"
        max="255"
        v-model="thresholdFirst"
      />

      <div class="colors-container">
        <table>
          <thead>
            <tr>
              <td>Основной</td>
              <td>Фон</td>
            </tr>
          </thead>
          <tbody>
            <tr>
              <td>
                <input
                  class="first-color"
                  type="color"
                  id="color"
                  v-model="firstColor"
                  placeholder="faf"
                />
              </td>
              <td>
                <input
                  class="second-color"
                  type="color"
                  id="color"
                  v-model="secondColor"
                />
              </td>
            </tr>
          </tbody>
        </table>
          
      </div>

      <div class="btns-container">
        <button class="cancel" @click="closeBinary">Вернуться</button>
        <button class="save-btn" @click="binaryProcess">Применить</button>
      </div>
    </div>


    <div v-if="isChangeBrightness" class="brightness-options">
      <span>Выберите яркость: {{ brightness }}</span>
      <input
        type="range"
        id="threshold"
        min="-255"
        max="255"
        v-model="brightness"
      />
      <span>Выберите контрастность: {{ contrast }}</span>
      <input
        type="range"
        id="threshold"
        min="0"
        max="3"
        step="0.1"
        v-model.number="contrast"
      />

      <div class="btns-container">
        <button class="cancel" @click="closeChangeBrightness">Вернуться</button>
        <button class="save-btn" @click="changeBrightnessContrastProcess">Применить</button>
      </div>
    </div>

    <div v-if="isNoiseOpen" class="brightness-options">
      <span>Выберите вероятность: {{ noiseProbability }}</span>
      <input
        type="range"
        id="threshold"
        min="0"
        max="0.4"
        step="0.1"
        v-model.number="noiseProbability"
      />

      <div class="btns-container">
        <button class="cancel" @click="closeAddNoise">Вернуться</button>
        <button class="save-btn" @click="addNoiseProcess">Применить</button>
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

.container .drawProfileLine{

  margin-top: 10px;

  width: 100%;
  height: 200px;

  display: flex;
  flex-direction: column;

  align-items: center;
  justify-content: center;

  background: #b1b1b1;
}

.container .drawProfileLine span{
  color: black;
  font-size: 22px;
  font-weight: 600;
  margin-bottom: 20px;
}

.container .drawProfileLine button{

  width: 150px;
  height: 60px;

  border-radius: 10px;
  background-color: white;

  font-size: 22px;

  cursor: pointer;
}

.container .profile {

margin-top: 20px;

align-items: center;
border: 1px solid #fff;

background-color: #b1b1b1;

width: 100%;
height: 300px;

display: flex;
flex-direction: column;
}

.container .profile canvas {
margin-top: 20px;
width: 90%;
}

.container .histograms {

margin-top: 10px;

width: 100%;

display: flex;
flex-direction: column;
}

.container .histograms .histsRow {

  margin-top: 10px;

  height: 120px;
  width: 100%;

  display: flex;
  flex-direction: row;

  justify-content: space-between;

  flex-wrap: wrap;
}

.container .histograms .histsRow .redHist, .greenHist, .blueHist, .intHist {

  align-items: center;
  border: 1px solid #fff;

  background-color: #b1b1b1;

  width: 220px;
  height: 100%;

  display: flex;
  flex-direction: column;
}

.container .histograms canvas {
  margin-top: 20px;
  width: 90%;
}

.container .btns-container {
  margin-top: 2px;
  height: 50px;
  width: 100%;
}

.container .btns-container button {

  all: unset;

  height: 50px;
  width: 50px;

  cursor: pointer;

  margin-right: 5px;
}

.container .btns-container .brightness-profile-btn {

background: url("/icon-profile.png");
background-size: contain;
}

.container .btns-container .graphs-btn {

background: url("/icon-hists.png");
background-size: contain;
}

.container .binary-options {

  position: fixed;
  top: 800px;
  left: 750px;

  height: 200px;
  width: 500px;

  background-color: #b2b2b2;

  display: flex;
  flex-direction: column;

  border-radius: 16px;

  align-items: center;

  z-index: 20;
}

.container .binary-options span {
  margin-top: 10px;
  margin-bottom: 10px;
}

.container .binary-options input {
  width: 70%;
  cursor: pointer;
}

.container .binary-options .colors-container {

  display: flex;
  flex-direction: row;

  margin-top: 20px;
  margin-bottom: 10px;

  gap: 40px;
}

.container .binary-options .colors-container .first-color{

  height: 40px;
  width: 140px;

  cursor: pointer;
}

.container .binary-options .colors-container .second-color{

height: 40px;
width: 140px;

cursor: pointer;
}

.container .binary-options .colors-container table td{
  text-align: center;
}

.container .binary-options .btns-container {

  display: flex;
  justify-content: center;
}

.container .binary-options .btns-container button{

  width: 140px;
  height: 30px;
  margin-right: 10px;
  margin-left: 10px;

  border-radius: 10px;

  background-color: #fff;

  cursor: pointer;

  text-align: center;
}

.container .brightness-options {

position: fixed;
top: 800px;
left: 750px;

height: 200px;
width: 500px;

background-color: #b2b2b2;

display: flex;
flex-direction: column;

border-radius: 16px;

align-items: center;

z-index: 20;
}

.container .brightness-options span {
  margin-top: 10px;
  margin-bottom: 10px;
}

.container .brightness-options input {
  width: 70%;
  cursor: pointer;
}

.container .brightness-options .btns-container {

  margin-top: 30px;
  display: flex;
  justify-content: center;
}

.container .brightness-options .btns-container button{

  width: 140px;
  height: 30px;
  margin-right: 10px;
  margin-left: 10px;

  border-radius: 10px;

  background-color: #fff;

  cursor: pointer;

  text-align: center;
}

.menu {

  width: 100vw;
  height: 70px;

  top: 0;
  left: 0;
  position: fixed;

  display: flex;

  align-items: center;

  background: #fff;

  z-index: 15;

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

  z-index: 10;
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

  cursor: pointer;
}

.menu-icon img, .toolbar-icon img{
  width: 96%;
  height: 96%;
  user-select: none;

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

  cursor: pointer;
}

.menu-icon img, .toolbar-selector img {
  width: 96%;
  height: 96%;
  user-select: none;

  overflow: hidden;
}

.toolbar-selector {
  
  position: relative;

  width: 50px;
  height: 50px;

  background-color: #fff;
  border: 1px solid #000000;
  border-radius: 4px;

  display: flex;
  justify-content: center;
  align-items: center;

  cursor: pointer;

  z-index: 10;
}

.toolbar-selector .filters {

  position: absolute;

  width: 260px;

  left: 48px;
  top: 0px;

  display: flex;
  flex-direction: column;
  text-align: left;

  gap: 4px;

  background-color: #000000;

  display: none;

  z-index: 15;
}

.toolbar-selector:hover .filters {
  display: flex;
  flex-direction: column;
}

.toolbar-selector .filters span {

  font-size: 16px;

  padding-top: 2px;
  padding-left: 10px;

  background-color: #2c2c2c;

  cursor: pointer;

  transition: .2s;
}

.toolbar-selector .filters span:hover {
  background-color: #000000;
}

.container {

  width: 90vw;
  
  padding: 50px;

  display: flex;
  flex-direction: row;

  justify-content: space-between;
}

.first-image-container {

  display: flex;
  flex-direction: column;

  align-items: center;

  position: relative;

  z-index: 5;
}

.second-image-container {

  display: flex;
  flex-direction: column;

  align-items: center; 
}

.image {

  border: 4px solid rgb(0, 0, 0);
  box-sizing: border-box;

  background-color: #b1b1b1;

  display: flex;
  flex-direction: column;

  position: relative;

  z-index: 5;
}

.image img {
  
  width: 100%;
  height: 100%;
  object-fit:contain;
}

.container h1 {
  margin-top: 10px;
  margin-bottom: 20px;

  color: #fff;
}

.crop-box {
  position: absolute;

  border: 2px solid rgba(0, 0, 0, 0.5);
  background-color: rgba(255, 255, 255, 0.089);
  cursor: move;
  z-index: 10;
}

.image .profile-line {

  position: absolute;
  height: 2px;
  background-color: red;
  border: 2px solid rgba(255, 0, 0, 0.5);

  z-index: 10;
}

.statement {

  position: absolute;

  width: 110px;
  height: 80px;

  display: flex;
  flex-direction: column;
  justify-content: center;
  padding-left: 10px;

  border: 1px solid black;
  border-radius: 10px;

  background-color: #d8d8d885;
  z-index: 15;
}

.green td{
  color: green;
}

.red td{
  color: red;
}

.blue td {
  color: blue;
}



</style>
