---
layout: post
title:  "[Survey] Visual Odometry and vSLAM"
date:   2019-07-17 09:00:01
categories: research
---





이 포스트는 Visual odometry (VO)와 Visual Simultaneous Localization and Mapping (vSLAM) 분야의 주요 논문들을 소스코드와 함께 분석하여 요약한다.  



# 1. ORB-SLAM2

<table>
<colgroup>
<col width="10%" />
<col width="90%" />
</colgroup>
<thead>
<tr class="header">
<th>제목</th>
<th>ORB-SLAM: A Versatile and Accurate Monocular SLAM System</th>
</tr>
</thead>
<tbody>
<tr>
<td markdown="span">저자</td>
<td markdown="span">Raul Mur-Artal, J. M. M. Montiel, and Juan D. Tardos</td>
</tr>
<tr>
<td markdown="span">출판</td>
<td markdown="span">TRO, 2015</td>
</tr>
<tr>
<td markdown="span">분류</td>
<td markdown="span">Relative scale, SLAM</td>
</tr>
<tr>
<td markdown="span">github</td>
<td markdown="span">https://github.com/raulmur/ORB_SLAM.git</td>
</tr>
</tbody>
</table>



<table>
<colgroup>
<col width="10%" />
<col width="90%" />
</colgroup>
<thead>
<tr class="header">
<th>제목</th>
<th>ORB-SLAM2: An Open-Source SLAM System for Monocular, Stereo,
and RGB-D Cameras</th>
</tr>
</thead>
<tbody>
<tr>
<td markdown="span">저자</td>
<td markdown="span">Raul Mur-Artal and Juan D. Tardos</td>
</tr>
<tr>
<td markdown="span">출판</td>
<td markdown="span">TRO, 2017</td>
</tr>
<tr>
<td markdown="span">분류</td>
<td markdown="span">Abolute scale, SLAM</td>
</tr>
<tr>
<td markdown="span">github</td>
<td markdown="span">https://github.com/raulmur/ORB_SLAM2.git</td>
</tr>
</tbody>
</table>



시작은 이 분야의 전설로부터 시작하는게 좋겠다. ORB feature로 SLAM의 A to Z를 완성시킨 **ORB SLAM**에서 절대 스케일을 알 수 있도록 Stereo, RGB-D 영상을 처리할 수 있게 발전시킨 논문이 **ORB SLAM2**다. 이 논문을 빼고 vSLAM이나 VO를 논할 수 없다. ORB SLAM2를 위주로 정리하되 ORB SLAM에서 물려받은 모듈들은 ORB SLAM 논문을 참고하여 설명하겠다.  



![orbslam](../assets/2019-07-17-savo-survey/orbslam1.png)



## Data

### Feature

당연한 이야기지만 ORB-SLAM(2)은 ORB feature를 쓴다. ORB는 "ORB: An efficient alternative to SIFT or SURF"란 논문으로 발표 되었다. ORB(Oriented FAST and Rotated BRIEF)라는 이름처럼 FAST corner detector와 회전에 강인한 rBRIEF descriptor를 조합한 feature다. ORB는 binary descriptor라서 추출과 비교가 빠르면서도 매칭 성능이 좋다.  

ORB-SLAM에서는 영상에서 ORB feature를 추출하여 tracking, mapping, relocalization, loop closing 등 feature가 필요한 모든 곳에 ORB를 쓴다.  

FAST keypoint는 프레임당 이미지 크기에 따라 1000 또는 2000 개를 뽑는다. 1.2배 단위로 8단계 이미지 스케일에서 뽑는다. 이미지에 keypoint들이 골고루 분포해야 하므로 그리드로 나눠서 cell 하나당 최소 5개 이상 나오도록 threshold를 조절한다.



### Keypoints

ORB SLAM2에서는 스테레오 영상에서 발견된 stereo keypoint를 $$\mathbf{x}_s=(u_L, v_L, u_R)$$ 형태로 관리한다. 스테레오 캘리브레이션이 완벽하게 되어 epipolar line이 수평하다면 왼쪽 이미지 좌표는 $$(u_L, v_L)$$ 일때 오른쪽 이미지 좌표는 $$(u_R, v_L)$$이 된다. RGB-D 영상인 경우 이를 스테레오 좌표로 변환한다. ($$f_x$$는 horizontal focal length, $$b$$는 baseline between projector and camera인데 어차피 가상의 스테레오를 만드는 것이니 적당히 아무값이나 넣어도 될듯 하다.)

$$
u_R=u_L-{f_xb \over d}
$$

이렇게 하면 스테레오와 RGB-D를 동일하게 처리할 수 있다. keypoint의 depth가 baseline의 40배 이하면 *close*로 분류하고 멀면 *far*로 분류한다. Close keypoint는 한 프레임 안에서 triangulate 하지만 far keypoint는 multi-view를 통해 여러번 봐야 triangulate 할 수 있다.  

단일 카메라에서 찾은 keypoint나 스테레오/RGB-D에서 depth를 찾을 수 없는 keypoint는 monocular keypoint $$\mathbf{x}_m=(u_L, v_L)$$로 정의한다. 이들도 multi-view를 통해 여러번 발견되어야 triangulate 할 수 있다.

불필요한 map point를 지우는 것을 **map point culling** 이라고한다. map point는 다음 조건을 만족해야 한다.

1. 어떤 map point가 보여지기로 기대되는 keyframe 중 25% 이상의 keyframe에서 실제로 보여야한다.
2. Map point는 3개 이상의 keyframe에서 발견되어야 한다.



코드에서 keypoint 가 가지고 있는 정보



### Keyframe

ORB SLAM에서 keyframe은 일단 자주 추가한 후 나중에 불필요한 keyframe을 지운다. keyframe은 다음 조건을 만족할 때 추가된다.

1. 마지막 global relocalization에서 20 프레임 이상 지났을 때
2. 마지막 keyframe을 추가한 후 20 프레임 이상 지났을 때
3. 현재 프레임이 최소 50 point를 추적할 때? (50 point 이하인듯)
4. 현재 프레임이 reference keyframe의 keypoint 중 90% 이하를 추적할 때
5. (ORB SLAM2) 추적 중인 close keypoint 수가 $$\tau_t=100$$개 이하로 떨어지고 현재 프레임에서 $$\tau_c=70$$개 이상의 close stereo keypoint를 만들어 낼 수 있을 때

Keyframe이 추가되면 covisibility graph에 새로운 노드를 추가하고 새 keyframe과 같은 keypoint들을 공유하는 기존 keyframe들을 연결한다.  

불필요한 keyframe을 지우는것을 **keyframe culling**이라고 한다. Keyframe culling policy는 어떤 keyframe의 90% 이상의 point들이 다른 세 개의 keyframe에서 보이면 그 keyframe을 지운다. Feature 매칭은 현재 이미지와 같거나 finer 스케일의 feature들과 매칭한다.



코드에서 keyframe 이 가지고 있는 정보





### Covisibility Graph

covisibility graph는 keyframe 사이의 관계를 나타낸 것으로 최소 15개의 map point를 공유한 keyframe들이 연결되고 공유 map point의 개수가 edge의 weight가 된다.





## Bundle Adjustment (BA)

ORB-SLAM2는 여러 단계에서 BA를 한다. BA는 g2o라는 비선형 최적화 라이브러리를 이용해 구현한다. 현재 프레임의 pose는 Motion-only BA, 최근 keyframe의 pose는 Local BA, 전체적인 최적화는 pose-graph optimization과 Full BA로 계산한다. 네 가지 최적화 모두 각각 다른 thread에서 독립적으로 돌아간다.



### A. Motion-only BA

Motion-only BA는 지도의 keypoint들은 고정시킨 채 현재 pose만 최적화한다. 아래의 reprojection error를 최소화하는 pose를 찾는다. (pose는 $$R \in SO(3), t \in \mathbb{R}^3$$ / keypoint 이미지 좌표는 $$\mathbf{x}_s, \mathbf{x}_m$$ / 지도의 3차원 keypoint 좌표는 $$\mathbf{X}$$)

![orbslam2](../assets/2019-07-17-savo-survey/orbslam2.png)

![orbslam3](../assets/2019-07-17-savo-survey/orbslam3.png)



### B. Local BA

새로운 keyframe이 생기면 covisible keyframes $$\mathcal{K}_L$$의 pose와 $$\mathcal{K}_L$$에서 보인 keypoint $$\mathcal{P}_L$$을 최적화한다. $$\mathcal{K}_L$$에 들어있지 않은 keyframe들의 pose도 최적화에 들어간 keypoint들을 봤다면 최적화에 사용되지만 고정된 파라미터로 들어간다.

![orbslam4](../assets/2019-07-17-savo-survey/orbslam4.png)



### C. Loop Closing and Full BA

Loop closing은 DBoW2로 발견하는데 발견하면 일단 pose-graph optimization을 한다. 즉 keypoint들은 계산에서 빼고 covisibility graph에서 pose 관계만 가지고 최적화를 하는 것이다. Pose-graph optimization이 끝나면 Full BA를 한다. Full BA가 끝나면 Full BA를 하는동안 추가된 keyframe, keypoint들을 업데이트한다.



## Code

### main

스테레오 카메라를 이용하는 경우를 위주로 코드를 분석해보자. 메인 함수는 `Examples` 폴더에 센서 별, 데이터셋 별로 따로 작성되어있다. 메인 함수에서는 `System` 클래스를 생성하고 `System::TrackStereo()`를 프레임마다 부른다.

```cpp
int main(int argc, char **argv)
{
    // ...
    ORB_SLAM2::System SLAM(argv[1],argv[2],ORB_SLAM2::System::STEREO,true);
    // ...
    for(int ni=0; ni<nImages; ni++)
    {
        imLeft = cv::imread(vstrImageLeft[ni],CV_LOAD_IMAGE_UNCHANGED);
        imRight = cv::imread(vstrImageRight[ni],CV_LOAD_IMAGE_UNCHANGED);
        SLAM.TrackStereo(imLeft,imRight,tframe);
    }
}
```

 

### System

`System` 클래스 생성자에서는 `LocalMapping`와 `LoopClosing` 클래스를 thread로 시작한다.  

 `System::TrackStereo()`에서는 `Tracking::GrabImageStereo()`를 부른다.

```cpp
// System.cc
System::System(...) :...
{
    //Load ORB Vocabulary
    mpVocabulary = new ORBVocabulary();
    bool bVocLoad = mpVocabulary->loadFromTextFile(strVocFile);

    //Create KeyFrame Database
    mpKeyFrameDatabase = new KeyFrameDatabase(*mpVocabulary);
    //Create the Map
    mpMap = new Map();

    //Initialize the Tracking thread
    mpTracker = new Tracking(this, mpVocabulary, mpFrameDrawer, mpMapDrawer,
                             mpMap, mpKeyFrameDatabase, strSettingsFile, mSensor);

    //Initialize the Local Mapping thread and launch
    mpLocalMapper = new LocalMapping(mpMap, mSensor==MONOCULAR);
    mptLocalMapping = new thread(&ORB_SLAM2::LocalMapping::Run,mpLocalMapper);

    //Initialize the Loop Closing thread and launch
    mpLoopCloser = new LoopClosing(mpMap, mpKeyFrameDatabase, mpVocabulary, mSensor!=MONOCULAR);
    mptLoopClosing = new thread(&ORB_SLAM2::LoopClosing::Run, mpLoopCloser);

    //Initialize the Viewer thread and launch
    if(bUseViewer)
    {
        mpViewer = new Viewer(this, mpFrameDrawer,mpMapDrawer,mpTracker,strSettingsFile);
        mptViewer = new thread(&Viewer::Run, mpViewer);
        mpTracker->SetViewer(mpViewer);
    }

    //Set pointers between threads
    mpTracker->SetLocalMapper(mpLocalMapper);
    mpTracker->SetLoopClosing(mpLoopCloser);

    mpLocalMapper->SetTracker(mpTracker);
    mpLocalMapper->SetLoopCloser(mpLoopCloser);

    mpLoopCloser->SetTracker(mpTracker);
    mpLoopCloser->SetLocalMapper(mpLocalMapper);
}


cv::Mat System::TrackStereo(const cv::Mat &imLeft, const cv::Mat &imRight, const double &timestamp)
{
	// 모드 확인 ...
    cv::Mat Tcw = mpTracker->GrabImageStereo(imLeft,imRight,timestamp);
	// ...
    return Tcw;
}
```



### Tracking

Tracking 코드는 다음과 같다. `Tracking::GrabImageStereo()` -> `Tracking::Track()` -> `Tracking::TrackReferenceKeyFrame()` -> `Optimizer::PoseOptimization()` 순서로 현재 프레임의 포즈를 추적한다.

```cpp
cv::Mat Tracking::GrabImageStereo(const cv::Mat &imRectLeft, const cv::Mat &imRectRight, const double &timestamp)
{
    mImGray = imRectLeft;
    cv::Mat imGrayRight = imRectRight;
	// 영상 gray 변환 ...
    // 프레임 생성
    mCurrentFrame = Frame(mImGray,imGrayRight,timestamp,mpORBextractorLeft,mpORBextractorRight,mpORBVocabulary,mK,mDistCoef,mbf,mThDepth);
    
    Track();
    
    return mCurrentFrame.mTcw.clone();
}

void Tracking::Track()
{
    if(mState==NOT_INITIALIZED)
        StereoInitialization();
    else
    {
        if(mState==OK)
        {
            // Local Mapping might have changed some MapPoints tracked in last frame
            CheckReplacedInLastFrame();
            bOK = TrackReferenceKeyFrame();
        }
        else
            bOK = Relocalization();

        // If we have an initial estimation of the camera pose and matching. Track the local map.
        if(!mbOnlyTracking)
            bOK = TrackLocalMap();

        // Check if we need to insert a new keyframe
        if(NeedNewKeyFrame())
            CreateNewKeyFrame();
    }
}

bool Tracking::TrackReferenceKeyFrame()
{
    // Compute Bag of Words vector
    mCurrentFrame.ComputeBoW();
    // ORB matching with the reference keyframe
    int nmatches = matcher.SearchByBoW(mpReferenceKF,mCurrentFrame,vpMapPointMatches);
    Optimizer::PoseOptimization(&mCurrentFrame);
}
```



### LocalMapping

`System` 생성자에서 `LocalMapping::Run()`을 위한 thread가 시작된다.

```cpp
void LocalMapping::Run()
{
    while(1)
    {
        // Tracking will see that Local Mapping is busy
        SetAcceptKeyFrames(false);

        // Check if there are keyframes in the queue
        if(CheckNewKeyFrames())
        {
            // BoW conversion and insertion in Map
            ProcessNewKeyFrame();
            // Check recent MapPoints
            MapPointCulling();
            // Triangulate new MapPoints
            CreateNewMapPoints();

            if(!CheckNewKeyFrames())
            {
                // Find more matches in neighbor keyframes and fuse point duplications
                SearchInNeighbors();
            }
            if(!CheckNewKeyFrames() && !stopRequested())
            {
                // Local BA
                if(mpMap->KeyFramesInMap()>2)
                    Optimizer::LocalBundleAdjustment(mpCurrentKeyFrame,&mbAbortBA, mpMap);

                // Check redundant local Keyframes
                KeyFrameCulling();
            }
            mpLoopCloser->InsertKeyFrame(mpCurrentKeyFrame);
        }
    }
}
```



### LoopClosing

`System` 생성자에서 `LoopClosing::Run()`을 위한 thread가 시작된다.

```cpp
void LoopClosing::Run()
{
    mbFinished =false;
    while(1)
    {
        // Check if there are keyframes in the queue
        if(CheckNewKeyFrames())
        {
            // Detect loop candidates and check covisibility consistency
            if(DetectLoop())
            {
               // Compute similarity transformation [sR|t]
               // In the stereo/RGBD case s=1
               if(ComputeSim3())
               {
                   // Perform loop fusion and pose graph optimization
                   CorrectLoop();
               }
            }
        }       
    }
}
```

