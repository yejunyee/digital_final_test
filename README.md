# 기말고사 답안지
1. ![image](https://user-images.githubusercontent.com/62829568/206953498-60cf4193-2bd8-4019-842a-90ff4266607e.png)

opencv활용
<pre>
<code>
let src = cv.imread('canvasInput'); 
let dst = cv.Mat.zeros(src.rows, src.cols, cv.CV_8UC3);
cv.cvtColor(src, src, cv.COLOR_RGBA2GRAY, 0); 
cv.threshold(src, src, 100, 200, cv.THRESH_BINARY); 
let contours = new cv.MatVector(); 
let hierarchy = new cv.Mat(); 
let poly = new cv.MatVector();
cv.findContours(src, contours, hierarchy, cv.RETR_CCOMP, cv.CHAIN_APPROX_SIMPLE); 
// approximates each contour to polygon
for (let i = 0; i < contours.size(); ++i) {
    let tmp = new cv.Mat();
    let cnt = contours.get(i);
    // You can try more different parameters
    cv.approxPolyDP(cnt, tmp, 5, true);
    poly.push_back(tmp);
    cnt.delete(); tmp.delete();
}
let color1 = new cv.Scalar(0,0,255,0); 
cv.drawContours(dst, poly, 3, color1, 16, 8, hierarchy, 0);
let color2 = new cv.Scalar(0,255,0,0); 
cv.drawContours(dst, poly, 4, color2, 16, 8, hierarchy, 0);
let color3 = new cv.Scalar(255,0,0,0); 
cv.drawContours(dst, poly, 2, color3, 16, 8, hierarchy, 0);
let color4 = new cv.Scalar(0,255,0,0); 
cv.drawContours(dst, poly, 1, color4, 16, 8, hierarchy, 0);
let color5 = new cv.Scalar(0,0,255,0); 
cv.drawContours(dst, poly, -1, color5, 16, 8, hierarchy, 0);
cv.imshow('canvasOutput', dst);
src.delete(); dst.delete(); hierarchy.delete(); contours.delete(); poly.delete();

</code>
</pre>

2.
이건 파이썬으로 진행했습니다
![image](https://user-images.githubusercontent.com/62829568/206961894-bd439e2d-3e15-4a16-b9a3-9c849897d68f.png)

<pre>
<code>
import cv2 as cv

def setLabel(image, str, contour):
    (text_width, text_height), baseline = cv.getTextSize(str, cv.FONT_HERSHEY_SIMPLEX, 0.7, 1)
    x,y,width,height = cv.boundingRect(contour)
    pt_x = x+int((width-text_width)/2)
    pt_y = y+int((height + text_height)/2)
    cv.rectangle(image, (pt_x, pt_y+baseline), (pt_x+text_width, pt_y-text_height), (200,200,200), cv.FILLED)
    cv.putText(image, str, (pt_x, pt_y), cv.FONT_HERSHEY_SIMPLEX, 0.7, (0,0,0), 1, 8)


img_color = cv.imread('finaltest.png', cv.IMREAD_COLOR)
cv.imshow('result', img_color)
cv.waitKey(0)

img_gray = cv.cvtColor(img_color, cv.COLOR_BGR2GRAY)
cv.imshow('result', img_gray)
cv.waitKey(0)

ret,img_binary = cv.threshold(img_gray, 127, 255, cv.THRESH_BINARY_INV|cv.THRESH_OTSU)
cv.imshow('result', img_binary)
cv.waitKey(0)

contours, hierarchy = cv.findContours(img_binary, cv.RETR_EXTERNAL, cv.CHAIN_APPROX_SIMPLE)

for cnt in contours:
    size = len(cnt)
    print(size)

    epsilon = 0.005 * cv.arcLength(cnt, True)
    approx = cv.approxPolyDP(cnt, epsilon, True)

    size = len(approx)
    print(size)

    cv.line(img_color, tuple(approx[0][0]), tuple(approx[size-1][0]), (0, 255, 0), 3)
    for k in range(size-1):
        cv.line(img_color, tuple(approx[k][0]), tuple(approx[k+1][0]), (0, 255, 0), 3)

    if cv.isContourConvex(approx):
        if size == 0:
            setLabel(img_color, "circle", cnt)
        elif size == 3:
            setLabel(img_color, "triangle", cnt)
        elif size == 4:
            setLabel(img_color, "rectangle", cnt)
        elif size == 5:
            setLabel(img_color, "pentagon", cnt)
        elif size == 6:
            setLabel(img_color, "hexagon", cnt)
        else:
            setLabel(img_color, str(size), cnt)
    else:
        setLabel(img_color, str(size), cnt)

cv.imshow('result', img_color)
cv.waitKey(0)

</code>
</pre>
2.  내 얼굴을 포함한 얼굴 인식
index.html
<pre>
<code>
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <meta http-equiv="X-UA-Compatible" content="ie=edge">
  <script defer src="face-api.min.js"></script>
  <script defer src="script.js"></script>
  <title>Face Recognition</title>
  <style>
    body {
      margin: 0;
      padding: 0;
      width: 100vw;
      height: 100vh;
      display: flex;
      justify-content: center;
      align-items: center;
      flex-direction: column
    }

    canvas {
      position: absolute;
      top: 0;
      left: 0;
    }
  </style>
</head>
<body>
  <input type="file" id="imageUpload">
</body>
</html>
</code>
</pre>
다음은 script.js
<pre>
<code>
const imageUpload = document.getElementById('imageUpload')

Promise.all([
  faceapi.nets.faceRecognitionNet.loadFromUri('/models'),
  faceapi.nets.faceLandmark68Net.loadFromUri('/models'),
  faceapi.nets.ssdMobilenetv1.loadFromUri('/models')
]).then(start)

async function start() {
  const container = document.createElement('div')
  container.style.position = 'relative'
  document.body.append(container)
  const labeledFaceDescriptors = await loadLabeledImages()
  const faceMatcher = new faceapi.FaceMatcher(labeledFaceDescriptors, 0.6)
  let image
  let canvas
  document.body.append('Loaded')
  imageUpload.addEventListener('change', async () => {
    if (image) image.remove()
    if (canvas) canvas.remove()
    image = await faceapi.bufferToImage(imageUpload.files[0])
    container.append(image)
    canvas = faceapi.createCanvasFromMedia(image)
    container.append(canvas)
    const displaySize = { width: image.width, height: image.height }
    faceapi.matchDimensions(canvas, displaySize)
    const detections = await faceapi.detectAllFaces(image).withFaceLandmarks().withFaceDescriptors()
    const resizedDetections = faceapi.resizeResults(detections, displaySize)
    const results = resizedDetections.map(d => faceMatcher.findBestMatch(d.descriptor))
    results.forEach((result, i) => {
      const box = resizedDetections[i].detection.box
      const drawBox = new faceapi.draw.DrawBox(box, { label: result.toString() })
      drawBox.draw(canvas)
    })
  })
}

function loadLabeledImages() {
  const labels = ['YeJun', 'YeungSoo', 'hyunWoo']
  return Promise.all(
    labels.map(async label => {
      const descriptions = []
      for (let i = 1; i <= 2; i++) {
        const img = await faceapi.fetchImage(`https://github.com/yejunyee/digital_final_test/tree/main/label_iamge`)
        const detections = await faceapi.detectSingleFace(img).withFaceLandmarks().withFaceDescriptor()
        descriptions.push(detections.descriptor)
      }

      return new faceapi.LabeledFaceDescriptors(label, descriptions)
    })
  )
}
</code>
</pre>
친구들이 지금 같이 모이기 힘들어서 사진을 비디오에 가져다가 인식되게 만들었습니다
