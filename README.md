# GrooveMet
Metronome for Android devices to visualize any rhythmic ideas.  
[GrooveMet on Google Play](https://play.google.com/store/apps/details?id=com.bzrt.groovemet&hl=en)   
![b](https://lh3.googleusercontent.com/bywa4sghZmpjx-GwM5cZi77rpOSqkNHKO1-G2P8OV_LNmPJKSK9qEiCjMi2QFvWihdjZ=w1440-h620-rw)
![a](https://lh3.googleusercontent.com/v_PDHhTvwelLqMw42281SWa3ClXmTMEynqhX5GbzdV8MTxDdhBER4KSNVWgr51AEWmg=w1440-h620-rw)  

Here I described how I implemented 1) Bezier curve and 2) fast flame effect.  

1) Bezier curve  
Include Matrix.java and reference below codes.  
- For the mathematic of matrix in Matrix.java, most of code was cited from [this link](http://introcs.cs.princeton.edu/java/95linear/Matrix.java.html) and I added some customization.  

```java
// we do not modify mArrPoint, but just generate mBezierArray from mArrPoint as raw data
private int BuildSmoothBezierArray() {
    //Log.d("SpaceView", "BuildSmoothBezierArray starts at :" + SystemClock.currentThreadTimeMillis());

    // set up the point arrays for input
    int iArraySize = mArrPoint.size();
    if (iArraySize ==0) return 0;
        /* // add 1 more Point at the end to create fully circular points array
        PointF[] cpArray = new PointF[iArraySize + 1];
        for (int i = 0; i < iArraySize; ++i)
            cpArray[i] = new PointF(mArrPoint.get(i).x, mArrPoint.get(i).y);
        cpArray[iArraySize] = new PointF(mArrPoint.get(0).x, mArrPoint.get(0).y);
        iArraySize = iArraySize + 1;
        // */
    PointF[] cpArray = new PointF[iArraySize];
    for (int i = 0; i < iArraySize; ++i)
        cpArray[i] = new PointF(mArrPoint.get(i).x, mArrPoint.get(i).y);

    //
    int msize = iArraySize - 2;
    PointF[] control = new PointF[iArraySize];
    for (int i = 0; i < iArraySize; ++i) control[i] = new PointF(0f, 0f);
    Matrix pMinv = new Matrix(msize, msize);
    for (int i = 0; i < msize; ++i) {
        for (int j = 0; j < msize; ++j) {
            if (i == j) pMinv.data[i][j] = 4;
            else if (i - j == -1 || i - j == 1) pMinv.data[i][j] = 1;
            else pMinv.data[i][j] = 0;
        }
    }
    pMinv = pMinv.inverse();
    Matrix pP = new Matrix(msize, 2);
    Matrix pB = new Matrix(msize, 2);

    //// control points
    // P
    pP.data[0][0] = (float) (6 * cpArray[1].x - cpArray[0].x);
    pP.data[0][1] = (float) (6 * cpArray[1].y - cpArray[0].y);
    pP.data[msize - 1][0] = (float) (6 * cpArray[iArraySize - 2].x - cpArray[iArraySize - 1].x);
    pP.data[msize - 1][1] = (float) (6 * cpArray[iArraySize - 2].y - cpArray[iArraySize - 1].y);
    for (int i = 1; i < msize - 1; ++i) {
        pP.data[i][0] = (float) (6 * cpArray[i + 1].x);
        pP.data[i][1] = (float) (6 * cpArray[i + 1].y);
    }
    // B
    pB = pMinv.times(pP);
    // Finally
    control[0] = cpArray[0];
    control[iArraySize - 1] = cpArray[iArraySize - 1];
    for (int i = 1; i <= msize; ++i) {
        control[i].x = Math.round(pB.data[i - 1][0]);
        control[i].y = Math.round(pB.data[i - 1][1]);
    }
    //
    int iAmpleBzCount = iArraySize * (mBezierDivNoBetween2Nodes + 2); // + 2 ? ; just for large enough array size
    PointF[] cpAmpleBzArray = new PointF[iAmpleBzCount];
    for (int i = 0; i < iAmpleBzCount; ++i) cpAmpleBzArray[i] = new PointF(0f, 0f);
    int iCount = 0;
    int x2, y2;
    float t;
    float lk = (float) 1 / (float) mBezierDivNoBetween2Nodes; //partition length  between 0.0 to 1.0
    for (int i = 1; i < iArraySize; ++i) {
        for (t = i - 1; t < i; t += lk) {
            if (iCount >= iAmpleBzCount) break;
            else if (t == i - 1) {
                // at least, orbit handle point must be included
                cpAmpleBzArray[iCount].x = cpArray[i - 1].x;
                cpAmpleBzArray[iCount].y = cpArray[i - 1].y;
                iCount++;
            } else {
                x2 = (int) ((float) cpArray[i - 1].x + (t - (i - 1)) * (-3 * (float) cpArray[i - 1].x +
                        3 * (.6667 * (float) control[i - 1].x + .3333 * (float) control[i].x) +
                        (t - (i - 1)) * (3 * (float) cpArray[i - 1].x - 6 * (.6667 * (float) control[i - 1].x +
                                .3333 * (float) control[i].x) + 3 * (.3333 * (float) control[i - 1].x +
                                .6667 * (float) control[i].x) + (-(float) cpArray[i - 1].x +
                                3 * (.6667 * (float) control[i - 1].x + .3333 * (float) control[i].x) -
                                3 * (.3333 * (float) control[i - 1].x + .6667 * (float) control[i].x) +
                                (float) cpArray[i].x) * (t - (i - 1)))));
                y2 = (int) ((float) cpArray[i - 1].y + (t - (i - 1)) * (-3 * (float) cpArray[i - 1].y +
                        3 * (.6667 * (float) control[i - 1].y + .3333 * (float) control[i].y) +
                        (t - (i - 1)) * (3 * (float) cpArray[i - 1].y - 6 * (.6667 * (float) control[i - 1].y +
                                .3333 * (float) control[i].y) + 3 * (.3333 * (float) control[i - 1].y +
                                .6667 * (float) control[i].y) + (-(float) cpArray[i - 1].y +
                                3 * (.6667 * (float) control[i - 1].y + .3333 * (float) control[i].y) -
                                3 * (.3333 * (float) control[i - 1].y + .6667 * (float) control[i].y) +
                                (float) cpArray[i].y) * (t - (i - 1)))));

                cpAmpleBzArray[iCount].x = x2;
                cpAmpleBzArray[iCount].y = y2;
                iCount++;
            }
        }
    }

    //OUTPUT  //////////////////////////////////////////////////////////////////////
    // bulk copy
    mTempBezierArray = null;
    mTempBezierArray = new PointF[iCount];
    for (int i = 0; i < iCount; ++i)
        mTempBezierArray[i] = new PointF(cpAmpleBzArray[i].x, cpAmpleBzArray[i].y);

    return iCount;
}
        
```

2) fast flame effect  
The principle of this effect was described in longbow games (http://www.longbowgames.com/particlefire/) long ago, which seemingly disappeared.  
In the nature, when dense substance exists on a spot, the substance spreads out and attenuates as time goes. We mimic that. 
At a point on screen, the RGB value attenuates as time goes and the decreased amount of RGB will be added to adjacent points. From the simple math, beautiful burn effect takes place on your Android phone. However, this consumes huge CPU and the screen lagging accumulated more and more on Android devices.  
The reason of the CPU consumption would be that the RGB array calculation is slow in java codes. So, a bitmap which caches screen image just ago was prepared in the memory and I called that a stamp. The stamp will be applied several times on current screen after adding more alpha value. RGB calculation will be done by the hardware more quickly than java codes.    

```java
// the stamp
private Paint mPaintStamp = new Paint();
mPaintStamp.setAntiAlias(true);
mPaintStamp.setAlpha(128);
mCanvasStamp.drawColor(Color.BLACK);
// mBurnEffectBasicSpacingInPx is the stamp spacing from the original position
mCanvasStamp.drawBitmap(mBitmapFinal, mBurnEffectBasicSpacingInPx, -mBurnEffectBasicSpacingInPx, mPaintStamp);
mCanvasStamp.drawBitmap(mBitmapFinal, -mBurnEffectBasicSpacingInPx, -mBurnEffectBasicSpacingInPx, mPaintStamp);
mCanvasStamp.drawBitmap(mBitmapFinal, mBurnEffectBasicSpacingInPx, mBurnEffectBasicSpacingInPx, mPaintStamp);
mCanvasStamp.drawBitmap(mBitmapFinal, -mBurnEffectBasicSpacingInPx, mBurnEffectBasicSpacingInPx, mPaintStamp);
// for upward flaming effect, applied another shot of stamp 
mPaintStamp.setAlpha(64);
mCanvasFinal.drawBitmap(mBitmapStamp, 0, -mBurnEffectUpwardSpacingInPx, mPaintStamp);
```

