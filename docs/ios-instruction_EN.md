# Agora Face Tracker SDK for iOS Quick Integration Guide 

Agora Face Tracker SDK includes the following features：

- Facial key points and face position detecting and tracking on static images
- Real-time detecting and tracking of 68 key points on face ( single face /multiple faces)
- Real-time filters, such as beautification and magic mirror
- Fun 2D stickers

Optimized for hardware occupancy, performance precision and efficiency of algorithm, the SDK can be used for live streaming on mobile devices, providing features like beautification camera, filter camera, fun stickers and simulation makeup, etc. 

## Flow Diagram

![](images/IOSpixelbuffer.png)

The following sections describe how to integrate the demo into your application, with some specific steps provided to help you use the demo. 

## Demo Quick Start Guide 

This demo implements face tracking and fun sticker features in a live broadcasting scenario based on Agora push stream. 

#### Environment Preparation

###### Requirements

  Software
    * XCode 6.0 or later
    * iOS 7.0 or later

  Hardware
    * Real mobile devices with voice and video functions


#### Complied Code Samples

1. Open the demo project file OpenLive in XCode. The following picture demonstrates an example of the directory structure:

  ![](images/ios-project.jpg)

2. Select the project and click "Build and Run" to compile codes. The following pictures show an example of the result.  

![](images/31.pic_hd.jpg) | ![](images/33.pic_hd.jpg) |

Note: The operation in this example only supports real devices and not simulators.

Run the demo when compilation completes.


## Integration Steps


#### Step 1: Prepare for the compilation environment

  Software
    * XCode 6.0 or later
    * iOS 7.0 or later

  Hardware
    * Real mobile devices with voice and video functions


#### Step 2: Add libraries 

  - __Provided by Agora__（obtained from SDK files）
    * libfaceTrackerSDK.a（the Face Tracker SDK）
    * libAGSDK.a（UI＋video rendering SDK）


  - __Download files__（you can skip this step if the files already exist in the project）

    Must have：
    *  	GPUImage video rendering with GPUImage SDK（使用GPUImage短视频录制必须使用Demo中的GPUImage SDK Use the GPUImage SDK in the demo for GPUImage short video recording）
  	https://github.com/BradLarson/GPUImage.git
    * opencv3.framework
  	https://sourceforge.net/projects/opencvlibrary/files/opencv-ios/3.0.0/

    Optional：
    * libyuv (Currently the SDK video rendering only supports NV21 format. Import video stream conversion class if video frame is YUV or any other video streaming type)
    https://github.com/lemenkov/libyuv.git


  - __System Library__ （included in xcode）
    * UIKit.framework
    * Foundation.framework
    * libz.tbd

#### Step 3: Import project 

1. Drag the AGFaceSDK folder to the project, and select "Create Groups" (Note: Use the GPUImage.a included in the demo if you need GPUImage video recording. Otherwise you won't need these library files.  )

2.  Import opencv3.framework

  Note: SDK downloaded from official website might be named as opencv2.framework by mistake.

#### Step 4: Configure stickers 

 Configure stickers if necessary. Sticker files are in the stickers folder with each sticker set under one directory, and config.json is included in every set of sticker files. The sticker file includes audio file, item parameter and some other information. See below for a sample structure: 

  ```
  |--[sticker_1] （Sticker 1）
  |   |--config.json （Sticker configuration file）
  |   |--[audio]（Audio file）
  |   |--[item_1]（Sticker item folder 1）
  |   |   |--[frame_1]（Sticker frame 1）
  |   |   |--[frame_2]（Sticker frame 2）
  |   |   |--...
  |   |   |--[frame_n]（Sticker frame n）
  |   |--[item_2]（Sticker item folder 2）
  |   |--...
  |   |--[item_n]（Sticker folder n）
  |--[sticker_2]（Sticker 2）
  |--...
  |--[sticker_n]（Sticker n）
  |—stickers.json（Sticker configuration file）
  ```
The application displays relevant stickers and icons by reading stickers.json under the StickerManager folder.

__Note__: To use stickers cloud, add a field “App Transport Security Settings” to Info.plist, and set YES for Allow Arbitrary Loads. 
The following tables list the parameters of a Json file: 

  __The following tables list the parameters of a Json file:__

  stickers.json

  Parameter name | Description
  ---------------|----------
  name | Name of the sticker
  dir | Name of the sticker folder
  category | Categories of sticker types
  thumb | File name of the sticker icon( under the same folder as voice file)
  voiced | true: with voice     false: no voice）
  downloaded | Indicates whether it has been downloaded or not. If not, the application can download it and then change the status here. 

  config.json

  Parameter name | Description
  ---------------|----------
  type | Types of the position the sticker will be displayed (e.g., face, full screen)
  facePos | Where the sticker goes on the face
  scaleWidthOffset | Scale ratio of the sticker width
  scaleHeightOffset | Scale ratio of the sticker height
  scaleXOffset | Offset coefficient of the sticker moving horizontally on face
  scaleYOffset | Offset coefficient of the sticker moving vertically on face
  alignPos | Alignment item parameter
  alignX | Offset coefficient of the edge moving horizontally
  alignY | Offset coefficient of the edge moving vertically
  frameFolder | Sticker resource folder ( including a set of picture frames)
  frameNum |  Frame numbers（a set of sequence frames makes an animation） 
  frameDuration | Interval of every frame（in second）
  frameWidth | Width of image
  frameHeight | Height of image
  trigerType | Trigger condition; by default it is set to 0; always displaying

  We provide a [Sticker Configuration website](https://apps.kiwiapp.mobi/sticker.html) for you to customize and generate the config.json file.

#### Step 5: Configure filters

Configure filters if necessary. Filter files are in the Filter folder with each filter set under one directory, and filter.png (filter lookUpTable) and thumb.png (filter icon) is included in every set of filter files. See below for a sample structure: 

  ```
  |--[filter_1] （filter 1）
  |   |--filter.png （filter lookUpTable）
  |   |--thumb.png（filter icon）
  |--[filter_2]（filter 2）
  |   |--thumb.png（filter icon）
  |--...
  |--[filter_n]（filter n）
  |—filters.json（filter config file）
  ```
Note: Filters whose resource files contain a filter.png（lookUpTable）are single-layer filters.

The application reads the filters.json file in the filter folder to display corresponding stickers and icons. 

  __The filters.json file format：__

| Parameter Name       | Description                                |
| ---------- | ---------------------------------     |
| name       | Name of the filter           |
| dir        | Name of the filter folder                   |
| category   | Categories of filter types	 |

#### Step 6:  Call APIs

###### UI Use built-in UI in the SDK

The SDK can be initialized in viewDidload if you choose to use the built-in UIs in the SDK.

```c
//1.Create AGRenderManager objects, and specify file path for models files. The default path is AGResource.bundle/models.
        renderManager = AGRenderManager.init(modelPath: nil, isCameraPositionBack: false)
    
//2.AGSDK Authentication prompt
        if (AGRenderManager.renderInitCode() != 0) {

            let alertView:UIAlertView = UIAlertView(title: "KiwiFaceSDK initialization failed. Error code: \(AGRenderManager.renderInitCode())", message: "check error code in FaceTracker.h", delegate: nil, cancelButtonTitle: "Cancel", otherButtonTitles: "OK")
            
            alertView.show()
            return;
        }
        
//3.Load stickers and filters
        renderManager.loadRender()
    
//4.Initialize UIManager
        AGSDKUI = AGUIManager(renderManager: renderManager, delegate: nil, superView: view)
    
//5. If use built-in UI, this function decides whether or not to clear the existing UI;  built-in UI may replace the existing UI if the latter does not have sufficient functions. 
        AGSDKUI.isClearOldUI = false
    
//6. Create built-in UI
        AGSDKUI .createUI()


 /* Render video frames. Called in every video proxy functions. */
[AGRenderManager processPixelBuffer:pixelBuffer]
/* 
Maximum faces to be tracked
default:4
1 <= maxFaceNumber <= 5
*/
self.renderManager. maxFaceNumber = 5;
```

###### 使用sdk自带功能 Use the SDK built-in funcitons


* Face key points and stickers：

  * Initialize face key points and stickers

    ```c
    self.pointsRender = [[AGPointsRenderer alloc]init];
    
    self.stickerRender = [[AGStickerRenderer alloc]init];
    ```
    
    * Call face key points and stickers
    
    ```c
	//call key points
	[self.renderer addFilter: self.pointsRender];
	
	//call stickers
	[self.renderer addFilter: self.stickerRender];
    ```
    
    * Remove face key points and stickers
    
    ```c
  //Remove key points
  [self.renderer removeFilter: self.pointsRender];
	
  //Remove stickers
  [self.stickerRender setSticker:nil];
    ```
* Magic mirror：
  * Initialize magic mirror filters

    ```c       
    self.distortionFilters = @[
         //square face
         [SquareFaceDistortionFilter new],
         //ET face
         [ETDistortionFilter new],
         //chubby face
         [FatFaceDistortionFilter new],
         //v-line face
         [SlimFaceDistortionFilter new],
         //Pear-shaped face
         [PearFaceDistortionFilter new]
    ];
    ```
   * Call magic mirror filters

    ```c
	//call magic mirror
  	[self.renderer addFilter:self.currentDistortionFilter];
	```

   * Remove magic mirror

  	```c
  //Remove magic mirror
  [self.renderer removeFilter:self.currentDistortionFilter];
	```
  
* Beautification：
  * Initialize beautification:

    ```c
    //Eye magnifying and facial slimming filter
    self.smallFaceBigEyeFilter = [[SmallFaceBigEyeFilter alloc]init];
    
    //Beautification filder
    self.beautyFilter = [[AGBeaytyFilter alloc]init];
    ```
  * Call beautification filters

    ```c
	//Call beautification filters
  	[self.renderer addFilter: self.beautyFilter];      
  	```
   * Remove beautification filters

  	```c
  //Remove beautification filters
  [self.renderer removeFilter: self.beautyFilter];
  	```

* General filters：
  * Call filters

  ```c
	//Call beautification filter
  	[self.renderer addFilter:self.currentColorFilter];     
  	```
  	
   * Remove filters
   
  ```c
  //Remove filters
    [self.renderer removeFilter:self.currentLookupFilter];  	```

###### Customized functions

* Add specific filters for rendering：

  he AGRenderer class in the SDK main class is used to add customized filters.
  
  ```
  [AGRenderManager.AGRenderer addFilter: GPUImageOutput<GPUImageInput, AGRenderProtocol> *];
  ```
  Filter objects must comply with the 2 protocols, namely GPUImageInput and FTRenderProtocol, to be tracked and rendered. 

* Remove specific filters and stop rendering: 

  ```
  [AGRenderManager.AGRenderer removeFilter: GPUImageOutput<GPUImageInput, AGRenderProtocol> *];
  ```
* When a face is captured and tracked, you can use customized block callback on every single frame before rendering: 

  ```
  typedef void (^RenderAndGetFacePointsBlock)(unsigned char *pixels, int format, int width, int height,result_68_t *p_result, int rstNum, int orientation,int faceNum);
  //block attributes
  @property (nonatomic, copy)RenderAndGetFacePointsBlock agRenderBlock;
  ```
  Block callbacks can be used before rendering and after face tracking. Use the following 3 parameters of Block to customize: 
  - result ：face coordinates (2D array; may contain multiple faces)
  - faceNum ：number of tracked faces
  - pixelBuffer：pixel stream of a single frame


###### Release memory

We recommend to always release memory before leaving a page.

```c
//Release all render objects
[self.renderManager releaseManager];
//Release UI
[self.AGUIManager releaseManager];
```
