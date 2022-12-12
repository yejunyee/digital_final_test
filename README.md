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
2. 
